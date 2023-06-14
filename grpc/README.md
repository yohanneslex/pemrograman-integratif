# pemrograman-integratif

## Persyaratan
Kita perlu pengetahuan dasar dari :
- Node JS
- Javasript
- MySQL
- Rest API
- Client to server communication

## Persiapan
Yang perlu diinstalasi :
- Node JS
- NPM

## Tahapan
### 1. Init Project
1.  Clone project
    ``` git clone https://github.com/yohanneslex/pemrograman-integratif ```
2.  Init code enc
    ```npm i```
    ```npm i grpc ```
    ```npm i mysql2 ```
    ``` npm i @grpc/proto-loader"```
3.  Buat 3 file
    - Proto file
    ```touch service_def.proto```
    - Server
    ```touch server.js```
    - Client
    ```touch client.js```
### 2. Fungsi CRUD
#### 1. Proto file
```proto
syntax = "proto3";

package User;

service UserService {
  rpc GetUser (GetUserRequest) returns (UserResponse) {}
  rpc AddUser (AddUserRequest) returns (UserResponse) {}
  rpc UpdateUser (UpdateUserRequest) returns (UserResponse) {}
  rpc DeleteUser (DeleteUserRequest) returns (UserResponse) {}
}

message GetUserRequest {
  string user_id = 1;
}

message AddUserRequest {
  string name = 1;
  int32 age = 2;
}
// 
message UpdateUserRequest {
  string user_id = 1;
  string name = 2;
  int32 age = 3;
}

message DeleteUserRequest {
  string user_id = 1;
}

message UserResponse {
  string name = 1;
  int32 age = 2;
}

```
#### 2. Server
```js
const grpc = require('grpc');
const protoLoader = require('@grpc/proto-loader');
const mysql = require('mysql2');

const PROTO_PATH = './grpc.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH);
const userProto = grpc.loadPackageDefinition(packageDefinition).User;

const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: '',
  database: 'tugaspi',
  waitForConnections: true,
  connectionLimit: 10,
});

function getUser(call, callback) {
  const userId = call.request.userId;
  pool.query('SELECT * FROM users WHERE user_id = ?', [userId], (error, results) => {
    if (error) {
      console.error(error);
      callback({ code: grpc.status.INTERNAL, message: 'Internal Server Error' });
      return;
    }
    if (results.length === 0) {
      callback({ code: grpc.status.NOT_FOUND, message: 'User not found' });
      return;
    }
    const userData = results[0];
    const user = {
      userId: userData.user_id,
      name: userData.name,
      age: userData.age,
    };
    callback(null, user);
  });
}

function addUser(call, callback) {
  const name = call.request.name;
  const age = call.request.age;
  pool.query('INSERT INTO users (name, age) VALUES (?, ?)', [name, age], (error, result) => {
    if (error) {
      console.error(error);
      callback({ code: grpc.status.INTERNAL, message: 'Internal Server Error' });
      return;
    }
    const userId = result.insertId;
    const user = { userId, name, age };
    callback(null, user);
  });
}

function updateUser(call, callback) {
  const userId = call.request.userId;
  const name = call.request.name;
  const age = call.request.age;
  pool.query('UPDATE users SET name = ?, age = ? WHERE user_id = ?', [name, age, userId], error => {
    if (error) {
      console.error(error);
      callback({ code: grpc.status.INTERNAL, message: 'Internal Server Error' });
      return;
    }
    const user = { userId, name, age };
    callback(null, user);
  });
}

function deleteUser(call, callback) {
  const userId = call.request.userId;
  pool.query('DELETE FROM users WHERE user_id = ?', [userId], error => {
    if (error) {
      console.error(error);
      callback({ code: grpc.status.INTERNAL, message: 'Internal Server Error' });
      return;
    }
    callback(null, {});
  });
}

function main() {
  const server = new grpc.Server();
  server.addService(userProto.UserService.service, {
    getUser,
    addUser,
    updateUser,
    deleteUser,
  });
  server.bindAsync('127.0.0.1:5000', grpc.ServerCredentials.createInsecure(), () => {
    console.log('Server running at http://127.0.0.1:5000');
    server.start();
  });
}

main();

```

#### 3. Client
```js
const grpc = require('grpc');
const protoLoader = require('@grpc/proto-loader');

const PROTO_PATH = './grpc.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH);
const userProto = grpc.loadPackageDefinition(packageDefinition).User;

const client = new userProto.UserService('localhost:5000', grpc.credentials.createInsecure());

function getUser(userId) {
  return new Promise((resolve, reject) => {
    client.getUser({ userId }, (error, user) => {
      if (error) {
        reject(error);
      } else {
        resolve(user);
      }
    });
  });
}


function addUser(name, age) {
  return new Promise((resolve, reject) => {
    const user = { name, age };
    client.addUser(user, (error, response) => {
      if (error) {
        reject(error);
      } else {
        resolve(response);
      }
    });
  });
}

function updateUser(userId, name, age) {
  return new Promise((resolve, reject) => {
    const user = { userId, name, age };
    client.updateUser(user, (error, response) => {
      if (error) {
        reject(error);
      } else {
        resolve(response);
      }
    });
  });
}

function deleteUser(userId) {
  return new Promise((resolve, reject) => {
    client.deleteUser({ userId }, (error, response) => {
      if (error) {
        reject(error);
      } else {
        resolve(response);
      }
    });
  });
}

async function test() {
  try {
    const user = await getUser(9);
    console.log('getUser:', user);

    const newUser = await addUser('Ronaldo', 38);
    console.log('addUser:', newUser);

    const updatedUser = await updateUser(1, 'Messi', 35);
    console.log('updateUser:', updatedUser);

    const result = await deleteUser(2);
    console.log('deleteUser:', result);
  } catch (error) {
    console.error(error);
  }
}

test();

```
## Output gRPC API
Gunakan dua terminal untuk menjalankan server.js ``` node server.js ``` dan client.js ``` node client.js ```
![image](https://user-images.githubusercontent.com/50076171/229402659-04874b72-5590-4b57-98a8-c172406ce7d6.png)
![image](https://user-images.githubusercontent.com/50076171/229402772-d8a34e4b-c4e2-423b-b262-e942cc1192f0.png)


