# Learn-node.js



#### install node.js in links

```
sudo apt-get update 
sudo apt-get install nodejs
node -v
```



## how to create new progect:
```
mkdir server
cd server
npm init
```

## how to install and use express
```
npm install express
```

###### code server in index.js in folder server this file run my server and this only ex:
```
const express = require('express')
const app = express()

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(3000, () => {
  console.log('Server started on port 3000')
})

```



###### how to run server :
run by node:
```
node index.js
```
or run by nodemon:
```
nodemon index.js
```

how to install nodemon:
```
npm instal nodemon
```
conf nodemon in file package.json:
```
"start": "nodemon app.js"
```





## how to create db:
``folder config code in file Database.js``
```
import { Sequelize } from "sequelize";
import dotenv from "dotenv";

dotenv.config();

const sequelize = new Sequelize(process.env.DATABASE_NAME, process.env.DATABASE_USER, process.env.DATABASE_PASSWORD, {
  host: process.env.DATABASE_HOST,
  port: process.env.DATABASE_PORT,
  dialect: "mysql",
  define: {
    timestamps: false,
  },
});

// Test the database connection
sequelize.authenticate()
  .then(() => console.log("Connection has been established successfully."))
  .catch((error) => console.error("Unable to connect to the database:", error));

// Sync the models with the database
// sequelize.sync({ force: true })
//   .then(() => console.log("Tables created successfully."))
//   .catch((error) => console.error("Error creating tables:", error));

export default sequelize;

```



## how to create table in db :
`` in folder Models to file tabal user add this code ! this only ex!!!  ``
```

import { Sequelize } from "sequelize";
import Database from "../../config/Database.js";
const { DataTypes } = Sequelize;

const Users = Database.define("users", {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
  password: DataTypes.STRING,
  refresh_token: DataTypes.TEXT,
  userId: DataTypes.STRING,
  image_user: DataTypes.STRING,
  country: DataTypes.STRING,
  flag: DataTypes.STRING,
  gender: DataTypes.STRING,
  user_name: DataTypes.STRING,
  first_name: DataTypes.STRING,
  last_name: DataTypes.STRING,
  university: DataTypes.STRING,
  sections: DataTypes.STRING,
  about: DataTypes.STRING,
  is_online: DataTypes.BOOLEAN,
  is_staff: DataTypes.BOOLEAN,
  is_active: DataTypes.BOOLEAN,
  is_assistant: DataTypes.BOOLEAN,
}, {
  freezeTableName: true,
});

// Users.sync();
export default Users;
```




## how to learn API :
###### config routers in file router
`GET`
```
router.get('/refreshToken', refreshToken);
```
`POST`
```
router.post('/login', Login);


```

`delete`
```
router.delete('/logout', Logout);
```

`PUT`
```
router.put('/logout', Logout);

```
`examle:`
```
import express from "express";
import multer  from 'multer';

import verifyToken from "../Views/verifyToken.js";
import getAllUsers from "../Views/getUsers.js";
import Register from "../Views/Register.js";
import Login from "../Views/login.js";
import refreshToken from "../Views/refreshToken.js";
import Logout from "../Views/Logout.js";
import updateUser from "../Views/user/updateUser.js";
import getUser from "../Views/user/getUser.js";

const router = express.Router();

router.get('/users', verifyToken , getAllUsers);
router.get('/user' , getUser);
router.post('/login', Login);
router.get('/refreshToken', refreshToken);
router.delete('/logout', Logout);





// user  

const storage = multer.diskStorage({
    destination:(req, file, cb) => {
        cb(null, './public')
    },
    filename:(req, file, cb) => {
        const fileName = `${Date.now()}_${file.originalname.replace(/\s+/g, '_')}`;
        cb(null , fileName);
    },
  })

const upload = multer({ storage }).single('image_user');

router.post('/uploads', upload, (req, res) => {
    const {file} = req;

    res.send({
        file:file.originalname,
        path:file.path
    });
})

router.post('/register', upload, Register);

router.put("/user/" , upload, updateUser);



export default router;


```








foe ex updetdata:
```
import bcrypt from "bcrypt";
import Users from "../../Models/UserModel.js";
import jwt from "jsonwebtoken";

const updateUser = async (req, res) => {
  try {
    const { name, email, image_user,  country, flag, gender, user_name, first_name, last_name, university, sections, about, is_online, is_staff, is_active, is_assistant } = req.body;
    const {token} = extractTokenFromHeader(req)
    const { id } = verifyToken(token)
    const user = await getUserById(id);


    const { file } = req;
    console.log("image_user",file && file.path);
    const updateObject = {};
    if (name) updateObject.name = name;
    if (email) updateObject.email = email;
    if (country) updateObject.country = country;
    if (flag) updateObject.flag = flag;
    if (gender) updateObject.gender = gender;
    if (file) updateObject.image_user = file && file.path || null;
    if (user_name) updateObject.user_name = user_name;
    if (first_name) updateObject.first_name = first_name;
    if (last_name) updateObject.last_name = last_name;
    if (university) updateObject.university = university;
    if (sections) updateObject.sections = sections;
    if (about) updateObject.about = about;
    if (is_online) updateObject.is_online = is_online;
    if (is_staff) updateObject.is_staff = is_staff;
    if (is_active) updateObject.is_active = is_active;
    if (is_assistant) updateObject.is_assistant = is_assistant;


    const data = await updateUserInDatabase(user , updateObject )
    return res.status(200).json({msg: "User Get successfully" , data:{
      id:user.id,
      name:user.name,
      email:user.email,
      refresh_token:user.refresh_token,
      image_user: user.image_user !== null ? req.protocol + '://' + req.get('host') + '/' + user.image_user : null,
      userId:user.userId,
      country:user.country,
      flag:user.flag,
      gender:user.gender,
      user_name:user.user_name,
      first_name:user.first_name,
      last_name:user.last_name,
      university:user.university,
      sections:user.sections,
      about:user.about,
      is_online:user.is_online,
      is_staff:user.is_staff,
      is_active:user.is_active,
      is_assistant:user.is_assistant,
      // وأي معلومات أخرى تريدها
    }});


  } catch (error) {
    console.error(error.message);
    return res.status(500).json({ msg: "Internal Server Error" });
  }
}

export default updateUser;


// استخراج التوكن من رأس الطلب
export const extractTokenFromHeader = (req) => {
  const authHeader = req.headers["authorization"];
  if (!authHeader) {
    throw new Error("Unauthorized: No token provided");
  }
  return {token: authHeader.split(" ")[1]};
};

// التحقق من صحة التوكن
export const verifyToken = (token) => {
  try {
    const decodedToken = jwt.verify(token, process.env.ACCESS_TOKEN_SECRET);
    return {id: decodedToken.userId};
  } catch (error) {
    throw new Error("Unauthorized: Invalid token");
  }
};

// الحصول على معلومات المستخدم باستخدام معرف المستخدم
export const getUserById = async (id) => {
  const user = await Users.findByPk(id);
  if (!user) {
    return res.status(404).json({ msg: "User not found" });
  }
  return user;
};

// تحديث معلومات المستخدم في قاعدة البيانات
export const updateUserInDatabase = async (user, updateObject) => {
  await user.update(updateObject);
  const updatedUser = await Users.findByPk(user.id);
  return updatedUser;
};



```
