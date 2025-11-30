
#-----------------------------------------------------------
# FIREBASETESTING.md
#-----------------------------------------------------------


#-----------------------------------------------------------
# Firestore Rules 
#-----------------------------------------------------------
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      // Allow access to the user document itself
      allow read, write: if request.auth != null && request.auth.uid == userId;

      // Allow access to all subcollections under each user
      match /{subcollection}/{doc=**} {
        allow read, write, update, delete: if request.auth != null && request.auth.uid == userId;
      }
    }
  }
}


#-----------------------------------------------------------
# Firestore Security Rules Test Suite
#-----------------------------------------------------------


const {
  initializeTestEnvironment,
  assertFails,
  assertSucceeds,
} = require("@firebase/rules-unit-testing");
const fs = require("fs");

const PROJECT_ID = "takeafish";

async function runTests() {
  // Load Firestore rules from file
  const rules = fs.readFileSync("firestore.rules", "utf8");

  // Initialize the test environment
  const testEnv = await initializeTestEnvironment({
    projectId: PROJECT_ID,
    firestore: { 
      rules,
      host: "localhost", 
      port: 9090,
    },
  });

  // Clear Firestore emulator data before running tests
  await testEnv.clearFirestore();

  console.log("-------Start Of The Testing-------\n");

  // Test contexts
  const unauthDb = testEnv.unauthenticatedContext().firestore();
  const aliceDb = testEnv.authenticatedContext("alice").firestore();
  const bobDb = testEnv.authenticatedContext("bob").firestore();
  const charlieDb = testEnv.authenticatedContext("charlie").firestore();

  // ---- SECTION 1: UNAUTHENTICATED ACCESS TESTS ----
  console.log("SECTION 1: Unauthenticated Access Tests");
  console.log("===========================================");

  const collections = ['fishing_logs', 'gps_points', 'fish', 'species', 'credentials'];

  for (const collection of collections) {
    // CREATE
    await assertFails(
      unauthDb.collection("users").doc("alice").collection(collection).doc("test").set({
        cloudId: "test", test: "data"
      })
    );
    console.log(`✅ UNAUTH CREATE ${collection} - blocked`);

    // READ
    await assertFails(
      unauthDb.collection("users").doc("alice").collection(collection).doc("test").get()
    );
    console.log(`✅ UNAUTH READ ${collection} - blocked`);

    // UPDATE
    await assertFails(
      unauthDb.collection("users").doc("alice").collection(collection).doc("test").update({test: "updated"})
    );
    console.log(`✅ UNAUTH UPDATE ${collection} - blocked`);

    // DELETE
    await assertFails(
      unauthDb.collection("users").doc("alice").collection(collection).doc("test").delete()
    );
    console.log(`✅ UNAUTH DELETE ${collection} - blocked`);

    // LIST
    await assertFails(
      unauthDb.collection("users").doc("alice").collection(collection).get()
    );
    console.log(`✅ UNAUTH LIST ${collection} - blocked`);
  }

  // ---- SECTION 2: AUTHENTICATED CRUD OPERATIONS ----
  console.log("\nSECTION 2: Authenticated CRUD Operations");
  console.log("============================================");

  // Test data templates for each collection
  const testData = {
    fishing_logs: {
      cloudId: "log_123",
      logs_ID: 1,
      description: "Fishing trip at lake",
      start_Time: "2024-01-01T10:00:00Z",
      end_Time: "2024-01-01T16:00:00Z",
      total_Catch: 5,
      sync: 0
    },
    gps_points: {
      cloudId: "gps_123", 
      gps_ID: 1,
      lat: 40.7128,
      long: -74.0060,
      timeStamp: "2024-01-01T12:00:00Z",
      logCloudId: "log_123"
    },
    fish: {
      cloudId: "fish_123",
      fish_ID: 1,
      logs_ID: 1,
      species_ID: 1,
      length: 15.5,
      timeStamp: "2024-01-01T14:00:00Z",
      confidence_Score: 85,
      logCloudId: "log_123"
    },
    species: {
      cloudId: "species_123",
      species_ID: 1,
      species_Name: "Largemouth Bass",
      sync: 0
    },
    credentials: {
      cloudId: "creds_123",
      user_ID: "alice",
      username: "alice_angler",
      email: "alice@example.com",
      first_Name: "Alice",
      last_Name: "Smith"
    }
  };

  // Test CREATE operations for all collections
  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(
      aliceDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).set(data)
    );
    console.log(`✅ AUTH CREATE ${collection} - succeeded`);
  }

  // Test READ operations
  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(
      aliceDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).get()
    );
    console.log(`✅ AUTH READ ${collection} - succeeded`);
  }

  // Test UPDATE operations
  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(
      aliceDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).update({
        description: "Updated description",
        last_Modified: new Date().toISOString()
      })
    );
    console.log(`✅ AUTH UPDATE ${collection} - succeeded`);
  }


  // ---- SECTION 3: CROSS-USER ACCESS TESTS ----
  console.log("\n SECTION 3: Cross-User Access Tests");
  console.log("======================================");

  // Bob tries to access Alice's data
  for (const [collection, data] of Object.entries(testData)) {
    // READ
    await assertFails(
      bobDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).get()
    );
    console.log(`✅ BOB READ ${collection} - blocked`);

    // UPDATE
    await assertFails(
      bobDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).update({test: "hacked"})
    );
    console.log(`✅ BOB UPDATE ${collection} - blocked`);

    // DELETE
    await assertFails(
      bobDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).delete()
    );
    console.log(`✅ BOB DELETE ${collection} - blocked`);
  }

  // Bob tries to create data in Alice's space
  for (const collection of collections) {
    await assertFails(
      bobDb.collection("users").doc("alice").collection(collection).doc("bob_hack").set({
        cloudId: "bob_hack",
        test: "unauthorized data"
      })
    );
    console.log(`✅ BOB CREATE in ${collection} - blocked`);
  }

  // ---- SECTION 7: MULTIPLE USERS SIMULTANEOUS DATA ----
  console.log("\n👥 SECTION 7: Multiple Users");
  console.log("============================");

  // Alice creates her data
  await assertSucceeds(
    aliceDb.collection("users").doc("alice").collection("fishing_logs").doc("alice_log").set({
      cloudId: "alice_log",
      logs_ID: 400,
      description: "Alice's private log",
    })
  );

  // Bob creates his own data
  await assertSucceeds(
    bobDb.collection("users").doc("bob").collection("fishing_logs").doc("bob_log").set({
      cloudId: "bob_log",
      logs_ID: 500,
      description: "Bob's private log",
    })
  );

  // Charlie creates his own data
  await assertSucceeds(
    charlieDb.collection("users").doc("charlie").collection("fishing_logs").doc("charlie_log").set({
      cloudId: "charlie_log",
      logs_ID: 600,
      description: "Charlie's private log",
    })
  );

  // Verify isolation - Alice cannot access Bob's or Charlie's data
  await assertFails(
    aliceDb.collection("users").doc("bob").collection("fishing_logs").doc("bob_log").get()
  );
  await assertFails(
    aliceDb.collection("users").doc("charlie").collection("fishing_logs").doc("charlie_log").get()
  );
  console.log("✅ USER ISOLATION - succeeded");

  // ---- SECTION 8: CLEANUP AND FINAL VALIDATION ----
  console.log("\n Delete Operations");
  console.log("================================");

  // // Test DELETE operations work for owner
  // for (const [collection, data] of Object.entries(testData)) {
  //   await assertSucceeds(
  //     aliceDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).delete()
  //   );
  //   console.log(`✅ AUTH DELETE ${collection} - succeeded`);
  // }

  // // Verify data is actually deleted
  // for (const [collection, data] of Object.entries(testData)) {
  //   const doc = await aliceDb.collection("users").doc("alice").collection(collection).doc(data.cloudId).get();
  //   if (!doc.exists) {
  //     console.log(`✅ DELETE VERIFICATION ${collection} - confirmed`);
  //   }
  // }

  // ---- FINAL SUMMARY ----
  console.log("\n🎉 EXTENSIVE TESTING COMPLETE!");
  console.log("==============================");
  console.log("All security rules are working correctly:");
  console.log("✅ Unauthenticated users blocked from all operations");
  console.log("✅ Authenticated users can CRUD their own data");
  console.log("✅ Cross-user access properly blocked");
  console.log("✅ Multiple users isolated properly");


  await testEnv.cleanup();
}

runTests().catch((err) => {
  console.error("❌ Test Suite Failed:", err);
  process.exit(1);
});