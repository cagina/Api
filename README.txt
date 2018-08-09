
var mysqlEasyModel = require('mysql-easy-model');
 
mysqlEasyModel.createConnection({
    connectionLimit : 10,
    host            : 'localhost',
    user            : 'root',
    password    : '',
    database        : ''
});

var User = mysqlEasyModel.model('user', {
  table: 'Tablo',
  fields: ['idTablo', 'Name','Surname', 'age'],
  primary: ['idTablo']
});

var User = mysqlEasyModel.model('user');






var express = require('express');
var app = express();
var bodyParser = require('body-parser');

app.use(bodyParser.urlencoded({ extended: true }));
app.use(bodyParser.json());

var server = app.listen(8081, function () {
   var host = server.address().address
   var port = server.address().port
   
   console.log("Example app listening at http://%s:%s", host, port)
})


var cors = require('cors');

// use it before all route definitions
app.use(cors());



//Listele

app.get('/users', function(req, res){
   User.query('SELECT * FROM ApiTest.Tablo', function(err, users){
    if(!err){

    if(users[0] != null){
    var response={};
    response.data=users;
    response.result={};
    response.result.id=parseInt(req.params.user_id);
    response.result.type="public"
    response.result.http="200 OK"
    res.send(response);
    }else{
      var response={};
      response.errors={};
      response.errors.source="users/id"
      response.errors.id=parseInt(req.params.user_id);
      response.errors.title="Query Parameter"
      response.errors.detail="NO Users"
      res.send(response);
    }
    }else{
      var response={};
      response.errors=[];
      response.errors.source="Connection"
      response.errors.title="Bad Connection"
      response.errors.detail="Connection Not Found"
    }
    
});


//Listeleme tek

app.get('/users/:user_id', function (req, res, next) {

  User.find({idTablo: parseInt(req.params.user_id)}, function(err, users){
    if(!err) {
    if (users[0] != null){
    var response={};
    response.data=users;
    response.result={};
    response.result.id=parseInt(req.params.user_id);
    response.result.type="public"
    response.result.http="200 OK"
    res.send(response);
    }else{
      var response={};
      response.errors={};
      response.errors.source="users/id"
      response.errors.id=parseInt(req.params.user_id);
      response.errors.title="Query Parameter"
      response.errors.detail="User Not Found"
      res.send(response);
    }
    }else{
     var response={
          "errors": [
        {
          source: "users/id" ,
          title:  "Query Parameter",
          detail: "User Not Found"
        }
          ]
            }
      res.send(response);
    }


   
  });
});
});



//update

app.put('/users/:user_id',function(req, res) {

    User.findOne({idTablo: parseInt(req.params.user_id)}, function(err, user){
    if(user){
        if(req.body.name)
        user.Name = req.body.name;
        if(req.body.surname)
        user.Surname = req.body.surname;
        if(req.body.age)
        user.age=req.body.age;
        user.update(function(err, result){
            if(!err)  res.send("Update");
        })
    }else
      res.send("Not found user");
});       
});



//create

app.post('/users/:user_id', function (req, res) {
    
    var user = new User({idTablo: parseInt(req.params.user_id),Name: req.body.name,Surname : req.body.surname ,age: req.body.age});
    user.create(function(err, result){
    if(!err)
    res.send("Create");
    else
    res.send("All ready exist");
});
})

//delete

app.delete('/users/:user_id', function (req, res) {
  
  
    User.findOne({idTablo: parseInt(req.params.user_id)}, function(err, user){
    if(user){
         user.destroy(function(err, result){
         if(!err) 
          res.send('Removed');
         });
    }else
    res.send("Not found user");
  }); 
});





app.get('/process_get', function (req, res) {
   
  var quer="select * from ApiTest.Tablo where";
  var arr=[]
  if(req.query.first_name){
  quer+=" Name= ?"
  arr.push(req.query.first_name);  
  }
  if(req.query.last_name){
  if(arr === undefined || arr.length == 0)
  quer+=" Surname= ?"
  else
  quer+=" and Surname= ?"
  arr.push(req.query.last_name);  
  }
  if(req.query.age){
  if(arr === undefined || arr.length == 0)
  quer+=" age=?"
  else 
  quer+=" and age=?"
  arr.push(req.query.age);  
  }
 User.query(quer, arr, function(err, users){
    if(users){
       if(!err) 
    res.send(users);
    }
    else
      alert("User Not Found");
  });
})
