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
