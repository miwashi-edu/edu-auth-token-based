# edu-auth-token-based

## Sequence Diagram

```mermaid
sequenceDiagram
    participant F as Frontend Server
    participant A as Auth Server
    participant B as Backend Server
    
    F->>A: Send login credentials
    A->>A: Verify credentials
    A-->>F: Send JWT if credentials are valid
    F->>F: Store JWT
    F->>B: Request data with JWT in Authorization Header
    B->>B: Validate JWT
    B-->>F: Send data if JWT is valid
```

## Frontend

### Prepare

```bash
# Set Up frontend Server
npx create-react-app frontend-server
cd frontend-server
npm install @babel/plugin-proposal-private-property-in-object
```

### .src/App.js

```
cat > src/App.js << 'EOF'
import React, { useState } from 'react';
import './App.css';

function App() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [data, setData] = useState(null);
  const [isAuthenticated, setAuthenticated] = useState(false);

  const handleLogin = async () => {
    try {
      const response = await fetch('http://localhost:3001/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      if (!response.ok) {
        throw new Error('Login failed');
      }

      const result = await response.json();
      localStorage.setItem('access_token', result.token);
      setAuthenticated(true);
    } catch (error) {
      alert(error.message || 'Login failed!');
    }
  };

  const handleLogout = () => {
    localStorage.removeItem('access_token');  // Remove token
    setAuthenticated(false);  // Update authentication state
    setData(null);  // Clear fetched data
  };

  const fetchData = async () => {
    try {
      const token = localStorage.getItem('access_token');
      const response = await fetch('http://localhost:3002/data', {
        headers: { Authorization: `Bearer ${token}` }
      });
      
      if (!response.ok) {
        if (response.status === 401) {  // Unauthorized
          handleLogout();
          alert('Session expired. Please login again.');
          return;
        }
        throw new Error('Failed to fetch data from backend');
      }

      const result = await response.json();
      setData(result.data);
    } catch (error) {
      alert(error.message || 'Failed to fetch data');
    }
  };

  return (
    <div className="App">
      <header className="App-header">
        {isAuthenticated ?
          <>
            {data ? <p>{data}</p> : <button onClick={fetchData}>Fetch Data</button>}
            <button onClick={handleLogout}>Logout</button>
          </> :
          (<div className="login-container">
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" className="input-field" />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" className="input-field" />
            <button onClick={handleLogin} className="login-button">Login</button>
          </div>)
        }
      </header>
    </div>
  );
}

export default App;
EOF
```

## ./src/App.css

```bash
cat > src/App.css << 'EOF'
.App {
  text-align: center;
}

.App-logo {
  height: 40vmin;
  pointer-events: none;
}

.App-header {
  background-color: #282c34;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  font-size: calc(10px + 2vmin);
  color: white;
}

.App-link {
  color: #61dafb;
}

.login-container {
  background-color: #444;
  border-radius: 5px;
  padding: 20px;
  max-width: 300px;
  margin: 0 auto;
}

.input-field {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  font-size: 16px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.login-button {
  background-color: #61dafb;
  color: #fff;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.login-button:hover {
  background-color: #21a1f1;
}

EOF
```

## Auth Server

### Prepare

```bash
mkdir auth-server && cd auth-server
mkdir -p ./src/routes
npm init -y
npm install express cors body-parser jsonwebtoken dotenv
npm install -D nodemon jest

npm pkg set main="./src/service.js"
npm pkg set scripts.start="node ./src/service.js"
npm pkg set scripts.dev="nodemon ./src/service.js"
npm pkg set scripts.test="jest"

# Create files
touch ./src/service.js ./src/server.js ./src/routes/auth_routes.js

```

### ./src/service.js

```bash
cat > ./src/service.js << 'EOF'
require('dotenv').config();
const app = require('./server.js');
const PORT = process.env.PORT || 3001

app.listen(PORT, () => {
    console.log(`http server listening on port ${PORT}`)
});
EOF
```

### ./src/server.js
```bash
cat > ./src/server.js << 'EOF'
const express = require('express');
const bodyParser = require('body-parser');
const authRoutes = require('./routes/auth_routes');
const cors = require('cors'); 

const app = express();

app.use(cors());  // Use CORS middleware without any restrictions
app.use(bodyParser.json());
app.use('/auth', authRoutes);

module.exports = app;
EOF
```

### ./src/routes/auth_routes.js
```bash
cat > ./src/routes/auth_routes.js << 'EOF'
const express = require('express');
const jwt = require('jsonwebtoken');

const router = express.Router();

router.post('/login', (req, res) => {
  const { email, password } = req.body;

  if (isValidUser(email, password)) {
    const token = jwt.sign({ email, role: getUserRole(email) }, 'secretKey', { expiresIn: '1h' });
    res.json({ token });
  } else {
    res.status(401).json({ error: 'Invalid login' });
  }
});

const isValidUser = (email, password) => {
  return email === 'user@example.com' && password === 'password';
};

const getUserRole = (email) => {
  return email === 'admin@example.com' ? 'admin' : 'user';
};

module.exports = router;
EOF
```


## Backend 

### Prepare

```bash
mkdir backend-server && cd backend-server
mkdir -p ./src/routes
npm init -y
npm install dotenv express cors body-parser jsonwebtoken
npm install -D nodemon jest

npm pkg set main="./src/service.js"
npm pkg set scripts.start="node ./src/service.js"
npm pkg set scripts.dev="nodemon ./src/service.js"
npm pkg set scripts.test="jest"

# Create files
touch ./src/service.js ./src/server.js ./src/routes/data_routes.js

```

### ./src/service.js

```bash
cat > ./src/service.js << 'EOF'
require('dotenv').config();
const app = require('./server.js');
const PORT = process.env.PORT || 3002

app.listen(PORT, () => {
    console.log(`http server listening on port ${PORT}`)
});
EOF
```

### ./src/server.js
```bash
cat > ./src/server.js << 'EOF'
const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const dataRoutes = require('./routes/data_routes');

const app = express();
app.use(cors());  // Use CORS middleware without any restrictions
app.use(bodyParser.json());
app.use('/data', dataRoutes);

module.exports = app;
EOF
```

### ./src/routes/data_routes.js
```bash
cat > ./src/routes/data_routes.js << 'EOF'
const express = require('express');
const jwt = require('jsonwebtoken');

const router = express.Router();

router.get('/', (req, res) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) return res.status(401).send('Access Denied');

  try {
    const verified = jwt.verify(token, 'secretKey');
    if (verified.role === 'admin') {
      res.json({ data: 'Secret data for admin!' });
    } else {
      res.json({ data: 'Secret data for user!' });
    }
  } catch {
    res.status(401).send('Invalid Token');
  }
});

module.exports = router;
EOF
```



