# js_back

    · npm init -y
    · npm i express mongoose body-parser jsonwebtoken bcrypt nodemon

  · package.json
  
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "dev": "nodemon index"
      },
  
  · app.js
  
    const express = require('express');
    const app = express();

    module.exports = app;
  
  · index.js
      
      const app = require('./app');

      const port = 3000;

      app.listen(port, () => {
        console.log(`Server running on port http://localhost:${port}`);
      });

      app.get('/', (req, res) => {
          res.send('Hello DBG');
      });      
      
      module.exports = app;

  · Create 4 folders

      config
      services
      controllers
      model
      routes


  · ./config/db.js
    
      const mongoose = require('mongoose');

      const connection = mongoose.createConnection(
          'mongodb://localhost:27017/TODO').on('open', () => {
              console.log('MongoDB connected');
          }).on('error', () => {
              console.log('MongoDB connection error');
          });
      ;
        
      module.exports = connection;          

 · Add to index.js

    const db = require('./config/db');




 · exit server by > ctrl D
 
 · run project > npm run dev




 · /model/user.model.js

    const mongoose = require('mongoose');
    const db = require('../config/db');
    
    const {Schema} = mongoose;
    
    const userSchema = new Schema({
        email : {
            type : String,
            lowercase : true,
            unique : true,
            required : true
        },
        password : {
            type : String,
            required : true
        }
    });
    
    const UserModel = db.model('user', userSchema);
    
    module.exports = UserModel;

 · Add to index.js

     const UserModel = require('./model/user.model');

 · Opem Mongo DB Compass and Connect

 · 
