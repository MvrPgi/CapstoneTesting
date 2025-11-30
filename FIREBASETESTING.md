
# FIREBASE TESTING SUITE



## Firestore Rules
```firestore
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
```


```javascript
const {
  initializeTestEnvironment,
  assertFails,
  assertSucceeds,
} = require('@firebase/rules-unit-testing');
const fs = require('fs');
const chalk = require('chalk');

const PROJECT_ID = 'takeafish';

const style = {
  header: (t) => chalk.bold.cyan('\n' + t + '\n' + '='.repeat(t.length)),
  pass: (t) => chalk.green(`✅ ${t}`),
  fail: (t) => chalk.red(`❌ ${t}`),
  info: (t) => chalk.blue(t),
  section: (t) => chalk.magenta.bold(`\n---- ${t} ----`),
};

async function runTests() {
  const rules = fs.readFileSync('firestore.rules', 'utf8');
  const testEnv = await initializeTestEnvironment({
    projectId: PROJECT_ID,
    firestore: { rules, host: 'localhost', port: 9090 },
  });

  await testEnv.clearFirestore();
  console.log(style.header('START FIRESTORE SECURITY TESTS'));

  const unauthDb = testEnv.unauthenticatedContext().firestore();
  const aliceDb = testEnv.authenticatedContext('alice').firestore();
  const bobDb = testEnv.authenticatedContext('bob').firestore();
  const charlieDb = testEnv.authenticatedContext('charlie').firestore();

  const collections = ['fishing_logs', 'gps_points', 'fish', 'species', 'credentials'];

  console.log(style.section('SECTION 1: Unauthenticated Access'));
  for (const collection of collections) {
    await assertFails(
      unauthDb.collection('users').doc('alice').collection(collection).doc('test').set({ cloudId: 'test', test: 'data' })
    );
    console.log(style.pass(`UNAUTH CREATE ${collection} blocked`));
    await assertFails(
      unauthDb.collection('users').doc('alice').collection(collection).doc('test').get()
    );
    console.log(style.pass(`UNAUTH READ ${collection} blocked`));
    await assertFails(
      unauthDb.collection('users').doc('alice').collection(collection).doc('test').update({ test: 'updated' })
    );
    console.log(style.pass(`UNAUTH UPDATE ${collection} blocked`));
    await assertFails(
      unauthDb.collection('users').doc('alice').collection(collection).doc('test').delete()
    );
    console.log(style.pass(`UNAUTH DELETE ${collection} blocked`));
    await assertFails(
      unauthDb.collection('users').doc('alice').collection(collection).get()
    );
    console.log(style.pass(`UNAUTH LIST ${collection} blocked`));
  }

  console.log(style.section('SECTION 2: Authenticated CRUD'));
  const testData = {
    fishing_logs: { cloudId: 'log_123', logs_ID: 1, description: 'Fishing trip at lake', start_Time: '2024-01-01T10:00:00Z', end_Time: '2024-01-01T16:00:00Z', total_Catch: 5, sync: 0 },
    gps_points: { cloudId: 'gps_123', gps_ID: 1, lat: 40.7128, long: -74.0060, timeStamp: '2024-01-01T12:00:00Z', logCloudId: 'log_123' },
    fish: { cloudId: 'fish_123', fish_ID: 1, logs_ID: 1, species_ID: 1, length: 15.5, timeStamp: '2024-01-01T14:00:00Z', confidence_Score: 85, logCloudId: 'log_123' },
    species: { cloudId: 'species_123', species_ID: 1, species_Name: 'Largemouth Bass', sync: 0 },
    credentials: { cloudId: 'creds_123', user_ID: 'alice', username: 'alice_angler', email: 'alice@example.com', first_Name: 'Alice', last_Name: 'Smith' },
  };

  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(aliceDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).set(data));
    console.log(style.pass(`AUTH CREATE ${collection}`));
  }
  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(aliceDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).get());
    console.log(style.pass(`AUTH READ ${collection}`));
  }
  for (const [collection, data] of Object.entries(testData)) {
    await assertSucceeds(
      aliceDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).update({ description: 'Updated description', last_Modified: new Date().toISOString() })
    );
    console.log(style.pass(`AUTH UPDATE ${collection}`));
  }

  console.log(style.section('SECTION 3: Cross-User Access'));
  for (const [collection, data] of Object.entries(testData)) {
    await assertFails(bobDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).get());
    console.log(style.pass(`BOB READ ${collection} blocked`));
    await assertFails(bobDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).update({ test: 'hacked' }));
    console.log(style.pass(`BOB UPDATE ${collection} blocked`));
    await assertFails(bobDb.collection('users').doc('alice').collection(collection).doc(data.cloudId).delete());
    console.log(style.pass(`BOB DELETE ${collection} blocked`));
  }
  for (const collection of collections) {
    await assertFails(bobDb.collection('users').doc('alice').collection(collection).doc('bob_hack').set({ cloudId: 'bob_hack', test: 'unauthorized data' }));
    console.log(style.pass(`BOB CREATE in ${collection} blocked`));
  }

  console.log(style.section('SECTION 7: Multiple Users Isolation'));
  await assertSucceeds(aliceDb.collection('users').doc('alice').collection('fishing_logs').doc('alice_log').set({ cloudId: 'alice_log', logs_ID: 400, description: "Alice's private log" }));
  await assertSucceeds(bobDb.collection('users').doc('bob').collection('fishing_logs').doc('bob_log').set({ cloudId: 'bob_log', logs_ID: 500, description: "Bob's private log" }));
  await assertSucceeds(charlieDb.collection('users').doc('charlie').collection('fishing_logs').doc('charlie_log').set({ cloudId: 'charlie_log', logs_ID: 600, description: "Charlie's private log" }));
  await assertFails(aliceDb.collection('users').doc('bob').collection('fishing_logs').doc('bob_log').get());
  await assertFails(aliceDb.collection('users').doc('charlie').collection('fishing_logs').doc('charlie_log').get());
  console.log(style.pass('USER ISOLATION confirmed'));

  console.log(style.header('EXTENSIVE TESTING COMPLETE'));
  console.log(style.info('All security rules behaved as expected.'));
  console.log(chalk.green('Unauthenticated users blocked')); 
  console.log(chalk.green('Authenticated users CRUD their own data')); 
  console.log(chalk.green('Cross-user access blocked')); 
  console.log(chalk.green('Multi-user isolation enforced'));

  await testEnv.cleanup();
}

runTests().catch((err) => {
  console.error(style.fail('Test Suite Failed'), err);
  process.exit(1);
});
```

