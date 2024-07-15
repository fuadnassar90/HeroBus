# HeroBus
Hero Bus is a mobile application created by Fuad Nassar and a talented team to solve Public transport issue in Jordan.

![image](https://github.com/user-attachments/assets/6bde1545-d449-4b78-81e3-b2a7ef94dc42)


---

# Server-Side Application

This server-side application is built using Node.js and Express.js. It incorporates Socket.IO for real-time communication, connects to a PostgreSQL database, and uses Firebase for authentication.

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   ```
2. Navigate to the project directory:
   ```bash
   cd <project-directory>
   ```
3. Install the dependencies:
   ```bash
   npm install
   ```

## Usage

To start the server, run:
```bash
npm start
```

## Important Parts of the Code

### Server Setup

The server uses Express.js to handle HTTP requests and Socket.IO for WebSocket connections.

```javascript
const express = require('express');
const socketIO = require('socket.io');
const PORT = process.env.PORT || 8080;
const INDEX = '/index.html';

const server = express()
    .use(express.urlencoded({ extended: true }), express.json())
    .use("/app", controller)
    .use("/", (req, res) => res.sendFile(INDEX, { root: __dirname }))
    .listen(PORT, () => console.log(`Listening on ${PORT}`));
const io = socketIO(server);
```

### Database Connection

The application connects to a PostgreSQL database using the `pg` package.

```javascript
const { Client } = require('pg');
const db = new Client({
    user: "your-db-user",
    password: "your-db-password",
    host: "your-db-host",
    port: "5432",
    database: "your-db-name",
    ssl: { rejectUnauthorized: false }
});
db.connect();
```

### Firebase Initialization

Firebase is used for authentication, initialized with a service account key.

```javascript
var admin = require("firebase-admin");
var serviceAccount = require("./serviceAccountKey.json");

admin.initializeApp({
    credential: admin.credential.cert(serviceAccount)
});
```

### Socket.IO Integration

The server handles WebSocket connections and disconnections, updating the database and broadcasting events to connected clients.

```javascript
io.on('connection', function(socket) {
    if(socket.handshake.query.id_route != 'routes' && socket.request.headers.id_route != null) {
        socket.id_auth = socket.request.headers.id + "";
        socket.id_route = socket.request.headers.id_route + "";
        socket.account_type = socket.request.headers.type + "";
        socket.lat = socket.request.headers.lat + "";
        socket.lng = socket.request.headers.lng + "";
        socket.rotate = socket.request.headers.rotate + "";
        socket.running = "moving";
        socket.speed = "0";

        console.log('New connection ' + socket.id + ' with type ' + socket.account_type + ' and id_auth ' + socket.id_auth);
        socket.join(socket.id_route);

        socket.on('disconnect', () => {
            handleDisconnection(socket);
        });

        socket.on('map_users', function (data) {
            handleMapUsersUpdate(socket, data);
        });

        socket.on('map_drivers', function (data) {
            handleMapDriversUpdate(socket, data);
        });

        initializeUserOrDriver(socket);
    } else {
        socket.join('routes');
        console.log('New connection for routes socket ' + socket.id);
        socket.on('disconnect', () => {
            console.log('Disconnected from routes socket ' + socket.id);
        });
    }
});
```

### Cron Jobs

A scheduled task runs every 4 hours to make a GET request to a specified URL.

```javascript
const cron = require('node-cron');
const https = require('https');

cron.schedule('0 */4 * * *', function() {
    console.log('Running a task every 4 hours');
    https.get('https://your-url.com/endpoint');
});
```

### Error Handling

The server handles uncaught exceptions to prevent crashes.

```javascript
process.on('uncaughtException', function (err, eew) {
    console.log('Error:', err.stack);
    if (!res.headersSent) {
        return res.send(500, { ok: false });
    }
    res.write('\n');
    res.end();
});
```

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

Feel free to customize this README file further based on your project's specific details and requirements.
