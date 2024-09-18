# BACKEND

    npm init -y
    npm i express mongoose body-parser jsonwebtoken bcrypt nodemon

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
        console.log(`Server running on port http://localhost:${port}`);
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

     const UserModel = require("../models/user.model");
    const jwt = require("jsonwebtoken");

    class UserServices{
     
        static async registerUser(email,password){
            try{
                    console.log("-----Email --- Password-----",email,password);
                    
                    const createUser = new UserModel({email,password});
                    return await createUser.save();
            }
            
            catch(e){
                throw e;
            }
        }
    
        static async getUserByEmail(email){
            try{
                return await UserModel.findOne({email});
            }catch(err){
                console.log(err);
            }
        }
    
        static async checkUser(email){
            try {
                return await UserModel.findOne({email});
            }
            
            catch (e) {
                throw e;
            }
        }
    
        static async generateAccessToken(tokenData,JWTSecret_Key,JWT_EXPIRE){
            return jwt.sign(tokenData, JWTSecret_Key, { expiresIn: JWT_EXPIRE });
        }
    }
    
    module.exports = UserServices;

 · ./controllers/user.controller.js

        const UserServices = require('../services/user.service');

        exports.register = async (req, res, next) => {
            try {
                console.log("---req body---", req.body);
                const { email, password } = req.body;
                const duplicate = await UserServices.getUserByEmail(email);

                if (duplicate) {
                    throw new Error(`UserName ${email}, Already Registered`)
                }
                
                const response = await UserServices.registerUser(email, password);
        
                res.json({ status: true, success: 'User registered successfully' });
                
            }
            
            catch (e) {
                console.log("---> e -->", err);
                next(e);
            }
        }

        exports.login = async (req, res, next) => {
            try {
        
                const { email, password } = req.body;
        
                if (!email || !password) {
                    throw new Error('Parameter are not correct');
                }
                
                let user = await UserServices.checkUser(email);

                if (!user) {
                    throw new Error('User does not exist');
                }
        
                const isPasswordCorrect = await user.comparePassword(password);
        
                if (isPasswordCorrect === false) {
                    throw new Error(`Username or Password does not match`);
                }
        
                let tokenData;
                tokenData = { _id: user._id, email: user.email };
            
                const token = await UserServices.generateAccessToken(tokenData,"secret","1h")
        
                res.status(200).json({ status: true, success: "sendData", token: token });
            } 
            
            catch (error) {
                console.log(error, 'err---->');
                next(error);
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

 
