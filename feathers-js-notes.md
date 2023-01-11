# App main commands
```bash
npm create feathers@pre feathers-chat
npm start
```
Next is creating something, evety command will give you dialog window do choose readable options:
```bash
npx feathers generate service # creates service with evetything what you need (methods and real time event handlers by interface)
npx feathers generate hook
```

Hooks are just middlewares. 

This framework smells like a dumb in case of here is no controllers or normal SOLID. Simplified piece of sht.


### IMPORTANT UPDATE

Very declarative and useful work with sockets. Maybe on this purpose it can be really useful.

# Endpoints

### Create user
```bash
curl 'http://localhost:3030/users/' \
  -H 'Content-Type: application/json' \
  --data-binary '{ "email": "hello@feathersjs.com", "password": "supersecret" }'
```
### Login 
```bash
curl 'http://localhost:3030/authentication/' \
  -H 'Content-Type: application/json' \
  --data-binary '{ "strategy": "local", "email": "hello@feathersjs.com", "password": "supersecret" }'
```

# Socket
Very declarative socket engine, which can be used on the most real-time projects, but we have to loop up to use seocket.io with React/Nest or React/Laravel|Python. I like them more.
