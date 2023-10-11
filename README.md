# edu-auth-token-based

# Testa Auth Server

## Förberedelser

```bash
npm install --save-dev jest supertest
```

## Test
```javascript
const request = require('supertest'); // För att göra HTTP-request i testerna
const app = require('../src/server.js'); // Importera din Express-app
```

## 1. Testa Registrering av Ny Användare
```javascript
test('POST /auth/register should create a new user', async () => {
  const newUser = { email: 'test@example.com', password: 'testpassword' };
  const response = await request(app).post('/auth/register').send(newUser);
  expect(response.statusCode).toBe(201);
  expect(response.body.email).toEqual(newUser.email);
});
```

## 2. Testa Inloggning med Rätt och Fel Uppgifter
```javascript
test('POST /auth/login with correct credentials should return a token', async () => {
  const user = { email: 'user@example.com', password: 'userpassword' };
  const response = await request(app).post('/auth/login').send(user);
  expect(response.statusCode).toBe(200);
  expect(response.body.token).toBeTruthy();
});

test('POST /auth/login with incorrect credentials should return 401', async () => {
  const user = { email: 'user@example.com', password: 'wrongpassword' };
  const response = await request(app).post('/auth/login').send(user);
  expect(response.statusCode).toBe(401);
});
```

## 3. Testa Token Generering och Validering
```javascript
test('Token should be generated on successful login', async () => {
  const user = { email: 'user@example.com', password: 'userpassword' };
  const response = await request(app).post('/auth/login').send(user);
  expect(response.body.token).toBeTruthy();
});

test('Token should be validated successfully', async () => {
  const response = await request(app).get('/auth/validate-token').set('Authorization', `Bearer ${token}`);
  expect(response.statusCode).toBe(200);
});
```

## 4. Testa Åtkomst med och utan giltig Token
```javascript
test('Request without or with invalid token should return 401', async () => {
  const responseWithoutToken = await request(app).get('/secure-endpoint');
  expect(responseWithoutToken.statusCode).toBe(401);
  
  const responseWithInvalidToken = await request(app).get('/secure-endpoint').set('Authorization', `Bearer invalid_token`);
  expect(responseWithInvalidToken.statusCode).toBe(401);
});

test('Request with valid token should return 200', async () => {
  const response = await request(app).get('/secure-endpoint').set('Authorization', `Bearer ${token}`);
  expect(response.statusCode).toBe(200);
});
```

# Cypress Tester för Frontend

## Förberedelser

```bash
npm install cypress --save-dev
npm pkg set scripts.cypress:open="cypress open"
```

## Test

## 1. Testa Att Sidan Laddas

```javascript
describe('Load page', () => {
  it('should load the login page successfully', () => {
    cy.visit('http://localhost:3000'); // Byt ut mot din app's URL
    cy.contains('Login'); // Ersätt med en textsträng som finns på din inloggningssida
  });
});
```

## 2. Testa Inloggning

```javascript
describe('Login', () => {
  it('should login successfully with correct credentials', () => {
    cy.visit('http://localhost:3000/login'); // Byt ut mot din app's URL
    cy.get('#email').type('user@example.com');
    cy.get('#password').type('userpassword');
    cy.get('#loginButton').click();
    cy.url().should('include', '/dashboard'); // Ersätt '/dashboard' med den URL som användaren omdirigeras till efter inloggning
  });
});
```

## 3. Testa Navigation mellan Sidor

```javascript
describe('Navigation', () => {
  it('should navigate between pages using navbar', () => {
    cy.visit('http://localhost:3000');
    cy.get('#navbarItem1').click();
    cy.url().should('include', '/page1');
    cy.get('#navbarItem2').click();
    cy.url().should('include', '/page2');
  });
});
```

## 4. Testa En Formulärinmatning

```javascript
describe('Form Submission', () => {
  it('should submit a form successfully', () => {
    cy.visit('http://localhost:3000/formpage'); // Byt ut mot din app's URL för formuläret
    cy.get('#textField').type('Sample Text');
    cy.get('#numberField').type('25');
    cy.get('#submitButton').click();
    cy.contains('Submission Successful'); // En sträng eller respons som bör visas vid lyckad inmatning
  });
});

```

## 5. Testa Respons på Felaktig Inmatning

```javascript
describe('Error Handling', () => {
  it('should display error message on invalid input', () => {
    cy.visit('http://localhost:3000/formpage'); // Byt ut mot din app's URL för formuläret
    cy.get('#textField').type('Invalid Input');
    cy.get('#submitButton').click();
    cy.contains('Please enter valid input'); // En felmeddelandetext som bör visas
  });
});

```

