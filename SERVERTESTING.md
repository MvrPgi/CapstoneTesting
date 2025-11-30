```python
# TEST SUITE FOR SERVERTESING.MD

import pytest
from server import app
from unittest.mock import patch
import tempfile
import flask

# -----------------------------------------------------------
# Test Suite for Image Upload Endpoint
# -----------------------------------------------------------
@pytest.fixture
def client():
  """Creates a Flask test client for simulating requests."""
  app.config["TESTING"] = True
  with app.test_client() as client:
    yield client

def dummy_image_file():
  """Creates a temporary dummy image file for upload tests."""
  temp_file = tempfile.NamedTemporaryFile(suffix=".jpg", delete=False)
  temp_file.write(b"dummy image data")
  temp_file.seek(0)
  return temp_file

def test_upload_image_success(client):
  """
  Test successful image upload.
  Mocks prediction and processing to simulate a valid fish detection.
  Expects 200 OK and correct response data.
  """
  with patch("server.predict_fish_specie") as mock_predict, \
     patch("server.process_prediction") as mock_process:
    mock_predict.return_value = {"predictions": [{"class_id": 0, "label": "TULINGAN", "confidence": 0.95}]}
    mock_process.return_value = {"message": "Success", "fish_detected": [{"species": "TULINGAN"}]}
    with dummy_image_file() as temp_img:
      data = {"image": (temp_img, "test.jpg")}
      response = client.post("/upload", data=data, content_type="multipart/form-data")
      assert response.status_code == 200
      assert response.json["message"] == "Success"
      assert response.json["fish_detected"][0]["species"] == "TULINGAN"

def test_upload_image_no_file(client):
  """
  Test upload with no file provided.
  Expects 400 Bad Request and appropriate error message.
  """
  response = client.post("/upload", data={}, content_type="multipart/form-data")
  assert response.status_code == 400
  assert response.json["error"] == "No Image File Provided"

def test_upload_image_unsupported_file_type(client):
  """
  Test upload with unsupported file type.
  Expects 400 Bad Request and appropriate error message.
  """
  with dummy_image_file() as temp_img:
    data = {"image": (temp_img, "test.txt")}
    response = client.post("/upload", data=data, content_type="multipart/form-data")
    assert response.status_code == 400
    assert response.json["error"] == "Unsupported file type"

def test_upload_image_file_too_large(client):
  """
  Test upload with file exceeding size limit.
  Expects 400 Bad Request and error about file size.
  """
  with tempfile.NamedTemporaryFile(suffix=".jpg", delete=False) as temp_img:
    temp_img.write(b"\xff\xd8\xff\xe0" + b"\x00" * (11 * 1024 * 1024))  # 11MB dummy data
    temp_img.seek(0)
    data = {"image": (temp_img, "test.jpg")}
    response = client.post("/upload", data=data, content_type="multipart/form-data")
    assert response.status_code == 400
    assert "File size exceeds" in response.json["error"]

def test_upload_image_empty_filename(client):
  """
  Test upload with empty filename.
  Expects 400 Bad Request and error about no file selected.
  """
  with dummy_image_file() as temp_img:
    data = {"image": (temp_img, "")}
    response = client.post("/upload", data=data, content_type="multipart/form-data")
    assert response.status_code == 400
    assert response.json["error"] == "No Image File Selected"

# -----------------------------------------------------------
# Test Suite for Fish Species and Coin Detection Functionality
# -----------------------------------------------------------

# Test: Successful fish species prediction
def test_predict_fish_specie_sucess():
  """
  Ensures predict_fish_specie returns correct prediction output
  when run_inference returns a valid fish prediction.
  """
  with patch('services.species.run_inference') as mock_run_inference:
    mock_run_inference.return_value = {
      "predictions": [{"class_id": 0, "label": "TULINGAN", "confidence": 0.95}]
    }
    result = predict_fish_specie("test_image.jpg")
    assert "predictions" in result
    assert result["predictions"][0]["label"] == "TULINGAN"
    assert result["predictions"][0]["confidence"] == 0.95
    assert "error" not in result

# Test: Processing prediction with fish and coin detected
def test_object_prediction():
  """
  Verifies process_prediction correctly processes fish and coin detection,
  converts dimensions, estimates age, and saves results.
  """
  with patch("services.species.detect_reference_coin") as mock_coin, \
     patch("services.species.convert") as mock_convert, \
     patch("services.species.estimate_age") as mock_age, \
     patch("services.species.save_to_sheets"):
    mock_coin.return_value = {"pixels_per_cm": 37.8, "coin_label": "10_PESO"}
    mock_convert.return_value = (2.5, 1.5)
    mock_age.return_value = {"days_before_maturity": 100}
    result = process_prediction(
      {"predictions": [{"detection_id": "1", "class": "TULINGAN", "confidence": 0.95, "width": 95, "height": 57}]},
      "test_image.jpg"
    )
    assert result["fish_detected"][0]["species"] == "TULINGAN"
    assert result["fish_detected"][0]["confidence"] == 0.95
    assert result["fish_detected"][0]["width_cm"] == 2.5
    assert result["fish_detected"][0]["height_cm"] == 1.5
    assert result["fish_detected"][0]["days_before_maturity"] == 100

# Test: detect_reference_coin returns correct message when no coin detected
def test_detect_reference_coin_no_predictions():
  """
  Checks detect_reference_coin returns appropriate message and zero pixels_per_cm
  when no coin predictions are found.
  """
  with patch("services.species.run_inference") as mock_run_inference:
    mock_run_inference.return_value = {"predictions": []}
    result = detect_reference_coin("test_image.jpg")
    assert result["message"] == "Walang Barya Na Nadetect"
    assert result["pixels_per_cm"] == 0

# Test: detect_reference_coin returns correct message for unknown coin
def test_detect_reference_coin_unknown_coin():
  """
  Ensures detect_reference_coin returns mapping failed message and None coin_label
  when coin is not in reference list.
  """
  with patch("services.species.run_inference") as mock_run_inference:
    mock_run_inference.return_value = {
      "predictions": [{"class_id": 99, "confidence": 0.85, "width": 50}]
    }
    result = detect_reference_coin("test_image.jpg")
    assert result["message"] == "Coin detected but mapping failed"
    assert result["pixels_per_cm"] == 0
    assert result["coin_label"] is None

# Test: run_inference returns error when no objects detected
def test_no_objects_detected():
  """
  Verifies run_inference returns error dictionary when no predictions are made.
  """
  with patch("services.utils.run_inference") as mock_run_inference:
    mock_run_inference.return_value = {"Error": "Walang Prediction"}
    result = services.utils.run_inference("test_image.jpg", "API_KEY", "MODEL_ID")
    assert result == {"Error": "Walang Prediction"}

# Test: estimate_age returns error for unknown species
def test_unknown_species():
  """
  Ensures estimate_age returns error message for unknown species.
  """
  with patch("services.species.estimate_age") as mock_estimate_age:
    mock_estimate_age.return_value = {"error": "Unknown species: UNKNOWN"}
    result = estimate_age(20, "UNKNOWN")
    assert result == {"error": "Unknown species: UNKNOWN"}

# Test: estimate_age returns valid result for known species
def test_estimate_age_known_species():
  """
  Checks estimate_age returns days_before_maturity for known species.
  """
  result = estimate_age(20, "TULINGAN")
  assert "days_before_maturity" in result
  assert isinstance(result["days_before_maturity"], (int, float))
  assert result["days_before_maturity"] <= 2002  

# Test: estimate_age returns zero for already mature fish
def test_estimate_age_already_mature():
  """
  Ensures estimate_age returns zero days_before_maturity for mature fish.
  """
  result = estimate_age(90, "TULINGAN")  # Assuming 50 cm is above maturity threshold
  assert result["days_before_maturity"] == 0

# Test: estimate_age returns positive value for negative length
def test_estimate_age_negative_length():
  """
  Checks estimate_age returns positive days_before_maturity for negative length input.
  """
  result = estimate_age(-5, "TULINGAN")
  assert "days_before_maturity" in result
  assert result["days_before_maturity"] > 0 

# Test: estimate_age returns positive value for zero length
def test_estimate_age_zero_length():
  """
  Ensures estimate_age returns positive days_before_maturity for zero length input.
  """
  result = estimate_age(0, "TULINGAN")
  assert "days_before_maturity" in result
  assert result["days_before_maturity"] > 0  
```
