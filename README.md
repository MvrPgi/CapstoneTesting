# This is a Capstone Project Testing Repository


## Testing Plans
This repository contains testing sheets for the Capstone Project. The sheets are designed to help ensure that all aspects of the project are thoroughly tested and validated. Refer to TESTCASES.md for detailed test cases and procedures to ensure functional testing


### Functional Testing Plans
Features to be Tested and the testing plans for each feature are as follows:


1. User Registration and Authentication
The application provides secure user account control for fishers that allows the creation and authentication of accounts for appropriate access to personal fishing records and system functionalities. This includes signup, login, and logout. A password recovery option also lets users reset credentials in case of forgotten passwords.

- Security Testing: Ensures user credentials and access levels are protected against unauthorized access. This plan is to ensure that the firebase authentication is properly implemented.
- Boundary Testing: Verifies input constraints like password length and character requirements to prevent invalid entries. This plan is to ensure the input fields are properly validated and properly implemented especially for our target audience.
- Usability Testing: Ensures a smooth login/signup process with intuitive navigation and accessible error messages. Ensure that the user interface is user-friendly and easy to navigate for our target audience.


2. Fish Identification via Object Detection 
The application automatically detects and classifies fish species from uploaded catch images via YOLOv11 Ultralytics, lessening manual identification errors through a simple click.

- Image Recognition Testing: Evaluates the accuracy of YOLOv11 Ultralytics in detecting and classifying fish species. This plan is to ensure that the model graph is properly implemented and the model is well-trained for our target fish species.

- Real World Scenario Testing: Tests the model's performance with various image qualities, angles to ensure robustness. This plan is to ensure that the model performs well in different scenarios that fishers might encounter in real life.



3. Fishing Route Tracking,Map View and Location Pinning 
The application enables users to record fishing routes and pinpoint catch locations to help fishers track movement patterns, optimize routes, and document their fishing spots efficiently. Additionally, location-based tagging allows for historical reference and navigation improvements, ensuring strategic fish management planning.

This feature provides an interactive visual representation of fishing locations, catch hotspots, and mapped routes. Users can track movement directly on the app. By leveraging real-time geospatial tracking, fishers gain enhanced decision-making capabilities for more efficient operations.

- GPS Accuracy Testing: Validates real-time tracking precision and ensures locations are accurately recorded. This is to ensure that the GPS tracking is properly implemented and accurate for our target audience.

- Geospatial Data Testing: Checks the accuracy of fishing routes and location markers on the map. 

- Load Testing: Ensures that mapping elements render properly without delays, even when handling multiple location inputs.


4. Fish Trend Analysis & Log History Records
The application gives basic analytics like fish caught by day and average size by day through report generation to provide fishers with insights into fish fluctuations and optimal fishing periods. This allows fishers to adjust strategies based on real-time data.

- Algorithm Validation Testing: Confirms simple SQL commands for accurate report generation on fish population trends.
- Data Integrity Testing: Ensures historical catch data is correctly stored, retrieved, and processed without corruption.


5. Error Handling 
- Input Validation Testing: Ensures all user inputs are validated to prevent incorrect data entries.
- Exception Handling Testing: Verifies that the application gracefully handles unexpected errors without crashing.


Each test case, whether automated or manual, was designed with specific inputs and expected outputs to validate the application's reliability. Success criteria were measured based on correctness, completeness, and robustness.





### Non-Functional Testing

1. Usability Testing

2. Compatibility Testing

3. Performance Testing

4. Security Testing