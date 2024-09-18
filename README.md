# BACKEND

    · npm init -y
    · npm i express mongoose body-parser jsonwebtoken bcrypt nodemon

  · package.json
  
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "dev": "nodemon index"
      },
  
  · app.js
  
    const express = require("express");
    const bodyParser = require("body-parser")
    const UserRoute = require("./routes/user.routes");
    const ToDoRoute = require('./routes/todo.router');
    const app = express();
    
    app.use(bodyParser.json())
    
    app.use("/",UserRoute);
    app.use("/",ToDoRoute);
    
    module.exports = app;
  
  · index.js

    const app = require("./app");
    const db = require('./config/db')

    const port = 3000;
    
    app.listen(port,()=>{
        console.log(`Server Listening on Port http://localhost:${port}`);
    })
      
    module.exports = app;

  · Create 4 folders

      config
      services
      controllers
      model
      routes


  · ./config/db.js
    
      const mongoose = require('mongoose');

      const connection = mongoose.createConnection(`mongodb://127.0.0.1:27017/ToDoDB`)
          .on('open',()=>{console.log("MongoDBConnected");})
              .on('error',()=>{
                  console.log("MongoDB Connection error");
              });
    
      module.exports = connection;          

**

 · exit server by > ctrl D
 
 · run project > npm run dev

**


 · ./model/user.model.js

    const db = require('../config/db');
    const bcrypt = require("bcrypt");
    const mongoose = require('mongoose');
    const { Schema } = mongoose;
    
    const userSchema = new Schema({
        email: {
            type: String,
            lowercase: true,
            required: [true, "userName can't be empty"],
           
            match: [
                /^([\w-\.]+@([\w-]+\.)+[\w-]{2,4})?$/,
                "userName format is not correct",
            ],
            unique: true,
        },

        password: {
            type: String,
            required: [true, "password is required"],
        },
    },{timestamps:true});
    
    userSchema.pre("save",async function(){
        var user = this;
        if(!user.isModified("password")){
            return
        }
        try{
            const salt = await bcrypt.genSalt(10);
            const hash = await bcrypt.hash(user.password,salt);
    
            user.password = hash;
        }catch(err){
            throw err;
        }
    });
    
    
    userSchema.methods.comparePassword = async function (candidatePassword) {
        try {
            console.log('----------------no password',this.password);
            // @ts-ignore
            const isMatch = await bcrypt.compare(candidatePassword, this.password);
            return isMatch;
        }
        
        catch (error) {
            throw error;
        }
    };
    
    const UserModel = db.model('user', userSchema);
    module.exports = UserModel;

 · Open Mongo DB Compass and Connect it

 · Create 3 files
     
     ./controller/user.controller.js
     ./routes/user.routes.js
     ./services/user.services.js

 · ./services/user.services.js

     const UserModel = require("../model/user.model");

     class UserService {
    
         static async registerUser(mail, password) {
             try {
                 const createUser = new UserModel({mail, password});
                 return await createUser.save();
             } 
            
             catch (e) {
                 throw e;
             }
         }
     }

     module.exports = UserService;

 · ./controllers/user.controller.js

    const  UserService = require('../services/user.services');

    exports.register = async(req, res, next) => {
        try {
            const  { email, password } = req.body;
            const successRes = await UserService.registerUser(email, password);
            res.json({status : true, success: 'User registred successfully!'})
        } 
        
        catch (e) {
            next(e);
        }
    }

 · ./routes/user.routes.js
    
    const router = require("express").Router();
    const UserController = require('../controller/user.controller');
    
    router.post("/register",UserController.register);
    
    router.post("/login", UserController.login);


module.exports = router;

 · Update app.js

    const express = require('express');
    const app = express();
    
    const bodyParser = require('body-parser');
    const userRouter = require('./routes/user.routes');
    
    app.use(bodyParser.json());
    
    app.use('/', userRouter);
    
    module.exports = app;

 · Update user.model.js to crypt the password

    const mongoose = require('mongoose');
    const db = require('../config/db');
    const bcrypt = require('bcrypt');
    
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
    
    userSchema.pre('save', async function() {
        try {
            var user = this;
            const salt = await (bcrypt.salt(10));
            const hashpwd = await bcrypt.hash(user.password, salt)
            user.password = hashpwd;
        } 
        
        catch (e) {
            next(e);
        }
    })
    
    const UserModel = db.model('user', userSchema);
    module.exports = UserModel;


     
