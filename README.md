# This is a Capstone Project Testing Repository
## Key Requirements

![alt text](image.png)

- **R1: Fish Detection**  
  The system must accurately detect and identify fish species from images.

- **R2: Real-Time GPS Tracking**  
  The application should provide real-time tracking of user locations during fishing activities.

- **R3: Data Synchronization**  
  Ensure seamless synchronization of user data between local storage and the cloud.

- **R4: Computer Vision Computation**  
  Integrate computer vision algorithms for image analysis and fish identification.

- **R5: Local Database**  
  Implement a local database to store user data and fishing records for offline access.

- **R6: Interactive Map Development**  
  Develop an interactive map feature for route tracking, location pinning, and visualization of fishing spots.

- **R7: Mobile Application Development**  
  Build the solution as a mobile application, ensuring compatibility with target devices.

- **R8: Offline Functionality**  
  The application must support core features and data access when offline.




## Testing Plans
This repository contains testing sheets for the Capstone Project. The sheets are designed to help ensure that all aspects of the project are thoroughly tested and validated. Refer to TESTCASES.md for detailed test cases and procedures to ensure functional testing

#### Testing Methodology

Each test case, whether automated or manual, was designed with specific inputs and expected outputs to validate the application's reliability. Success criteria were measured based on correctness, completeness, and robustness.

Tests are categorized by level and scope:
- **Unit Testing:** Validates individual modules in isolation
- **Integration Testing:** Verifies interactions between components
- **Functional Testing:** Confirms features meet requirements through end-to-end workflows
- **System Testing:** Comprehensive evaluation of the entire application


--- 

### Unit Testing Plans

#### 1.1 Authentication Module
- **Testing Approaches:** Test Firebase authentication methods (signup, login, logout, password reset) with mocked Firebase calls to verify correct handling of success and error states.
**Test Implementation:** See [FIREBASETESTING.md](FIREBASETESTING.md)

#### 1.2 Firestore Database Module
- **Testing Approaches:** Use Firebase Rules Unit Testing library to simulate authenticated and unauthenticated access to Firestore documents. Validate CRUD operations against defined security rules. 
**Test Implementation:** See [FIREBASETESTING.md](FIREBASETESTING.md)

#### 1.3 Fish Species Detection Module
- **Testing Approaches:**  YOLOv11 Ultralytics model inference calls to test the fish detection logic. Validate that the correct species are identified based on predefined input images.
- **Image Recognition Testing:** Evaluates the accuracy of YOLOv11 Ultralytics in detecting and classifying fish species or reference images. This plan is to ensure that the model graph is properly implemented and the model is well-trained for our target fish species.
- **Real World Scenario Testing:** Tests the model's performance with various image qualities, angles to ensure robustness. This plan is to ensure that the model performs well in different scenarios that fishers might encounter in real life.
- **Size Estimation Testing:** Validates the accuracy of fish size measurements based on reference coin detection. This plan is to ensure that the size estimation algorithm is properly implemented and accurate for our target audience.

**Test Implementation:** See [SERVERTESTING.md](SERVERTESTING.md)


--- 

### Integration Strategy 

The project follows an **MVP-first incremental integration** approach:

1. **MVP Phase Testing** Core modules (fish detection, datasync, database, GPS tracking) were integrated and tested first to ensure foundational functionality.
2. **Incremental Feature Testing** As new features were developed, targeted tests were added to validate their integration with existing systems.
3. **Regression Testing** After each development cycle, regression tests ensured that new changes did not adversely affect existing functionalities.

### Integration Testing Plans

Validates the interactions between modules using MVP-first incremental integration:


### Phase 1: MVP Integration (Core Functionality)
**Modules Integrated:** Fish Detection 

**Test Scenario**
- Upload an image and verify fish species detection.
- Size estimation based on fallback parameters.

**Success Criteria**
- Correct species identified.
- Accurate size estimation within acceptable error margins because of missing reference coin.

### Phase 2: GPS Tracking Integration
**Modules Integrated:** GPS Tracking, Local Database.

**Test Scenario**
- Start a fishing session and track location.
- Save GPS points to local database.

**Success Criteria**
- GPS points accurately recorded and stored.

### Phase 3: API Integration
**Modules Integrated:** Flask API Endpoint Integrations

**Test Scenario**
- Upload images via API and receive detection results.
**Success Criteria**
- API responds with correct detection data.

### Phase 4: Local Only Database and Authentication
**Modules Integrated:** Local Database, Authentication Module, Log History Module.

**Test Scenario**
- User signup/login and data storage locally.
- Add and retrieve fishing logs.

**Success Criteria**
- User can authenticate and access local data.

### Phase 6: Other Module Integrations
**Modules Integrated:** Map View, Analytics Module.

**Test Scenario**
- View fishing routes on map and generate trend reports.

**Success Criteria**
- Routes displayed correctly and reports generated accurately.

### Phase 7: Coin Detection Integration
**Modules Integrated:** Coin Detection, Fish Detection.

**Test Scenario**
- Upload image with reference coin and verify size estimation.
**Success Criteria**
- Accurate size estimation using detected coin dimensions.

### Phase 8: Firebase Authentication & Cloud Sync Integration 
**Modules Integrated:** Cloud Sync, Firestore Database, Authentication Module.

**Test Scenario**
- Sync local data with Firestore and verify consistency.
- Login with Firebase Authentication and access cloud data.

**Success Criteria**
- Data matches between local and cloud databases.
---

### Functional Testing Plans


This testing plan is designed to validate the core functionalities of the Capstone Project application. This is done when the development of each feature is completed or updated. 

Features to be Tested and the testing plans for each feature are as follows:

#### 1. User Registration and Authentication

The application provides secure user account control for fishers that allows the creation and authentication of accounts for appropriate access to personal fishing records and system functionalities. This includes signup, login, and logout. A password recovery option also lets users reset credentials in case of forgotten passwords.

**Testing Approaches:**
- **Security Testing:** Ensures user credentials and access levels are protected against unauthorized access. This plan is to ensure that the Firebase authentication is properly implemented.
- **Boundary Testing:** Verifies input constraints like password length and character requirements to prevent invalid entries. This plan is to ensure the input fields are properly validated and properly implemented especially for our target audience.
- **Usability Testing:** Ensures a smooth login/signup process with intuitive navigation and accessible error messages. Ensure that the user interface is user-friendly and easy to navigate for our target audience.



#### 2. Fish Identification via Object Detection

The application automatically detects and classifies fish species from uploaded catch images via YOLOv11 Ultralytics, lessening manual identification errors through a simple click.

**Testing Approaches:**
- **Data Validation Testing:** Ensures uploaded images meet format and size requirements to prevent processing errors. This is to ensure that the image upload functionality is properly implemented and robust for our target audience.


#### 3. Fishing Route Tracking, Map View and Location Pinning

The application enables users to record fishing routes and pinpoint catch locations to help fishers track movement patterns, optimize routes, and document their fishing spots efficiently. Additionally, location-based tagging allows for historical reference and navigation improvements, ensuring strategic fish management planning.

This feature provides an interactive visual representation of fishing locations, catch hotspots, and mapped routes. Users can track movement directly on the app. By leveraging real-time geospatial tracking, fishers gain enhanced decision-making capabilities for more efficient operations.

**Testing Approaches:**
- **GPS Accuracy Testing:** Validates real-time tracking precision and ensures locations are accurately recorded. This is to ensure that the GPS tracking is properly implemented and accurate for our target audience.
- **Geospatial Data Testing:** Checks the accuracy of fishing routes and location markers on the map.
- **GPS Comparison Testing:** Compares recorded GPS data against known locations to verify accuracy.
- **Load Testing:** Ensures that mapping elements render properly without delays, even when handling multiple location inputs.

#### 4. Fish Trend Analysis & Log History Records

The application gives basic analytics like fish caught by day and average size by day through report generation to provide fishers with insights into fish fluctuations and optimal fishing periods. This allows fishers to adjust strategies based on real-time data.

**Testing Approaches:**
- **Algorithm Validation Testing:** Confirms simple SQL commands for accurate report generation on fish population trends.
- **Data Integrity Testing:** Ensures historical catch data is correctly stored, retrieved, and processed without corruption.

#### 5. Error Handling

**Testing Approaches:**
- **Input Validation Testing:** Ensures all user inputs are validated to prevent incorrect data entries.
- **Exception Handling Testing:** Verifies that the application gracefully handles unexpected errors without crashing.


#### 6. Firebase Security Rules

**Testing Approaches:**
- **Access Control Testing:** Validates that users can only access their own data and cannot access others' data.
- **CRUD Operations Testing:** Ensures that authenticated users can create, read, update, and delete their own records as per the defined security rules.



### System Testing

Comprehensive end-to-end testing of the entire application to ensure all components work together seamlessly.

#### 1. End-to-End Testing
- **Testing Approaches:** Simulate real-world user scenarios that encompass all major functionalities of the application.

**Test Scenarios:**
- **Complete Fishing Session Workflow:**
  1. User logs in with valid credentials
  2. Starts GPS tracking for fishing route
  3. Captures fish image and detects species
  4. Adds catch to log with location tag
  5. Views catch on map with marker
  6. Stops tracking and saves route
  7. Views analytics showing updated trends
  8. Syncs data to cloud (if online)
  9. Logs out successfully

  **Success Criteria:**
  - Each step completes without errors
  - Data is accurately recorded and displayed


#### 2. Usability & Reliability Testing
- **Testing Approaches:** Conduct a testing for No Cellular Network mode, Offline Mode, and general navigation to ensure ease of use for fishers.

**Test Scenarios:**
- **No Cellular Network Mode:** Verify application functionality when GPS signal is weak or unavailable.
- **Offline Mode:** Test core features ( log history access,load mapping,gps points) without internet connectivity.
- **GPS Availability:** Ensure GPS tracking works accurately in various cellphone states(close,idle,active).

**Success Criteria:**
- Application remains functional and user-friendly in all tested scenarios excluding cloud sync and image recognition.
- Navigation is intuitive for target users

#### 3. Performance Testing
- **Testing Approaches:** Measure application responsiveness during key operations such as image upload, fish detection, GPS tracking, and data synchronization.

**Test Scenarios:**
- **Image Upload and Detection:** Measure time taken from image upload to fish species identification.
- **GPS Tracking:** Evaluate responsiveness of real-time location updates during active tracking.
- **Data Synchronization:** Assess time taken to sync local data with cloud database under varying network conditions.
**Success Criteria:**
- All operations complete without lag or crashes
- Battery consumption remains within acceptable limits during prolonged use (30 minutes of continuous GPS tracking)

#### 5. Compatibility Testing 
- **Testing Approaches:** Validate application functionality across a range of Android devices and Versions to ensure consistent user experience.

- **Test Scenarios:**
- **Device Variety:** Test on multiple Android devices with different screen sizes and hardware capabilities.
- **OS Versions:** Test on various Android versions (e.g., Android 9.0 to the latest) to ensure compatibility.
**Success Criteria:**
- Application functions correctly on all tested devices and OS versions without UI glitches or performance issues.


#### 6. Regression Testing
- **Testing Approaches:** After each major system update or bug fix, all previously validated features are re-tested to ensure that recent changes have not introduced new issues. This process includes verifying the continued accuracy of fish identification, the stability of GPS tracking, and the reliability of analytics and forecasting features. Regression testing helps maintain overall application stability and ensures that enhancements or fixes do not negatively impact existing functionality.

- **Level-Based Approach:** Regression testing is prioritized based on the complexity of potential fixes:
  - **Easy:** Minor fixes or UI adjustments that are unlikely to affect other parts of the application. These are quickly tested to confirm no unintended side effects.
  - **Medium:** Moderate changes such as updates to business logic or integration points, which may impact multiple features. These require more thorough testing across related modules.
  - **Hard:** Major changes or architectural updates that could affect core workflows or data integrity. These undergo comprehensive regression testing across the entire system to ensure overall stability.

**Success Criteria:**
- No previously functioning features are broken after updates.
- The BUGS is resolved if not isolated further testing is required.

#### 7. Priority Testing

- **Testing Approaches:** Critical functionalities that directly impact user experience and data integrity, such as user authentication, fish species detection, and GPS tracking, are prioritized for testing. These features undergo more frequent and rigorous testing cycles to ensure they meet high standards of reliability and performance. 

**Success Criteria:**
- Critical features consistently perform as expected under various conditions.
- Any issues falls under priority features are Leveled High and addressed with proper team meetings.


#### 8. Security Testing
- **Testing Approaches:** Ensure that all data authetication, access, and data trasmission are secure and protected against unauthorized access.

**Test Scenarios:**
- **Authentication Security:** Verify that only authenticated users can access their data.


---

### Document and Code Review

#### Document Review

Document review ensures that all project documentation is complete, consistent, and accurate. The following documents were reviewed:

- **System Requirements Specification (SRS):** Checked for clear, testable, and stakeholder-aligned functional and non-functional requirements.
- **Design Documents:** Assessed diagrams (e.g., data flow, fishbone, system architecture) for accuracy and completeness.
- **Test Plans and Test Cases:** Validated for comprehensive coverage, logical structure, and alignment with system objectives.
- **User Manuals and Training Materials:** Reviewed for clarity, accuracy, and suitability for the target audience, including non-technical users such as barangay volunteers.

#### Code Review

Code review is a key process to maintain code quality, enhance performance, and prevent security issues. Review activities included:

- **Peer Review:** The team reviewed each other's code before integration to catch logical errors, enforce coding standards, and identify inefficiencies.
- **Static Code Analysis:** Utilized tools or IDE analyzers to detect syntax errors, unused variables, and potential memory leaks. Such as GITHUB Copilot or other linters.
- **Version Control Review:** Code commits were managed via GitHub pull requests, enabling transparent tracking of changes and referencing related issues or enhancements. 
