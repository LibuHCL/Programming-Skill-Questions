# MSTS programming skill answers :

The below has the pseudo code for the relevant questions.

### Table of Contents :

| No. | Questions |
|---- | ---------
|1 | [How to increase error visibility in NodeJs with an example](#How to increase error visibility in NodeJs with an example)|
|2 | [How to parse a json if the json is very big using Node.js](#How to parse a json if the json is very big using Node.js)|
|3 | [How to extract data which is in json format through http request using node js with an example](#How to extract data which is in json format through http request using node js with an example)|
|4 | [How do you calculate the time taken by function to run in node js with an example](#How do you calculate the time taken by function to run in node js with an example)|
|5 | [Create a download progress bar in nodejs. there are 5 steps in the progress bar, each taking 10 secs to complete.](#Create a download progress bar in nodejs there are 5 steps in the progress bar each taking 10 secs to complete.)|
|6 | [How to automatically generated the next id in Mongo db using spring boot with an example](#How to automatically generated the next id in Mongo db using spring boot with an example)|
|7 | [Name any key management tool/framework to can manage the internationalization which can be integrated with the microservice](#Name any key management tool/framework to can manage the internationalization which can be integrated with the microservice)|
|8 | [How can I convert a list into Binary in perl with an example](#How can I convert a list into Binary in perl with an example)|
|9|  [How to include 2 html pages inside the html page using angularjs with an example](#How to include 2 html pages inside the html page using angularjs with an example)|

1. ### How to increase error visibility in NodeJs with an example.

    Errors are thrown from the system, If the application doesnt handles the exception been thrown from the code. Following the 
    best practices like using promises for asynchronous error handling, Using built in error object, Shutting down the process
    gracefully, Using mature logger like Morgan, Winston, Bunyan, Node-Loggly or Log4J will speed up the error visibility for the developers and understand
    the root cause of the issue avoiding the console.log which is cumbersome to identify.
    
    Below is a pseudo code example for which I've used only winston(multi-transport async logging library for Node.js) and morgan(HTTP request logging middleware) for logging the statements according 
    to the debug levels.
    
    npm install winston morgan --save 
    
    **index.js** :
    
    const express = require ('express'),
        morgan = require('morgan');
    
    const PORT = 3000;
    
    const app = express();
    
    app.use(morgan('PROD', {
        skip: function (req, res) {
            return res.statusCode < 400
        }, stream: process.stderr
    }));
    
    app.use(morgan('PROD', {
        skip: function (req, res) {
            return res.statusCode >= 400
        }, stream: process.stdout
    }));
    
    app.get('/', function (req, res) {
      res.send('Hello World!')
    })
    
    app.listen(PORT, function(){
        console.log('Example app listening on port ' + PORT);
    });
    
    **logger.js** :
  
    const winston = require("winston");
  
    const level = process.env.LOG_LEVEL || 'debug';
  
    const logger = new winston.Logger({
      transports: [
          new winston.transports.Console({
              level: level,
              timestamp: function () {
                  return (new Date()).toISOString();
              }
          })
          new (winston.transports.File)({ filename: 'testfile.log' })
      ]
    });
  
    module.exports = logger
    
   Consolidated **index.js** :
   
   const express = require ('express'),
       morgan = require('morgan'),
       logger = require('./logger');
   
   const PORT = 3000;
   
   const app = express();
   
   app.use(morgan('PROD', {
       skip: function (req, res) {
           return res.statusCode < 400
       }, stream: process.stderr
   }));
   
   app.use(morgan('PROD', {
       skip: function (req, res) {
           return res.statusCode >= 400
       }, stream: process.stdout
   }));
   
   app.get('/', function (req, res) {
       logger.debug('Debug statement');
       logger.info('Info statement');
       res.send('Hello World!');
   });
   
   app.use(function(req, res, next){
       logger.error('404 page requested');
       res.status(404).send('This page does not exist!');
   });
   
   app.listen(PORT, function(){
       logger.info('Example app listening on port ' + PORT);
   });
   
   Running the server using the node index.js command and call this route then the output on our console would be like this -
   
   2019-05-25T07:43:26.638Z - debug: Debug statement
   2019-05-25T07:43:26.639Z - info: Info statement
   
2. ## How to parse a json if the json is very big using Node.js ##

  In order to handle this scenario there are couple of ways we can achieve It.
  
  * Splitting a large file into smaller files might speed up things if they're read asynchronously or in parallel
   (for example by using worker threads). To process a file line-by-line, simply need to decouple the reading of the file 
   and the code that acts upon that input. This can be accomplished  by buffering the input until we hit a newline.
  * Reading the file data in smaller chunks, for example by:
    * Buffering data line-by-line like by using the readline module or looking for the newline character.
    * Using a JSON stream parsing module.
   
   It would be something like below.
   
   var JSONStream = require("JSONStream");
   var es = require("event-stream");
   
   fileStream = fs.createReadStream(filePath, { encoding: "utf8" });
   fileStream.pipe(JSONStream.parse("testfile.*")).pipe(
     es.through(function(data) {
       console.log("printing file object read from file ::");
       console.log(data);
       this.pause();
       processOneCustomer(data, this);
       return data;
     }),
     function end() {
       console.log("stream reading ended");
       this.emit("end");
     }
   );
   
   function processOneCustomer(data, es) {
     DataModel.save(function(err, dataModel) {
       es.resume();
     });
   }
   
   http.request(options, function(res) {
     var stream = JSONStream.parse('*')
     res.pipe(stream)
     stream.on('data', console.log.bind(console, 'an item'))
   })
   
3. ## How to extract data which is in json format through http request using node js with an example ##

    The best way to read the Json data is to use **JSON.parse()** passing string as input. The below is the 
    asynchronous version of that.
  
  var fs = require('fs');
  
  fs.readFile('/path/to/file.json', 'utf8', function (err, data) {
      if (err) throw err; // we'll not consider error handling for now
      var obj = JSON.parse(data);
  });
  
  But since the question is to retrieve from request object the below POST example holds good.
  
   var express = require("express");
   var myParser = require("body-parser");
   var app = express();
   app.use(myParser.json({extended : true}));
    app.post("/path to the resource", function(request, response) {
         console.log(request.body);
   });
   
   app.listen(8080);
   
4. ## How do you calculate the time taken by function to run in node js with an example ##

  Obviously we set the time before the start of the function and after the function is execution is completed we
  get the time and calculate the time taken by the difference of both.
  
  var start = new Date();
  // Rest of the code goes here
  var end = new Date() - start;
  console.log("Execution time: ", end);
  
  There is also an alternative way in nodeJS which provides a function called hrtime to get high resolution timings.
  
  Below is an example of that.
  
  var start = new Date();
  var hrstart = process.hrtime();
  
  setTimeout(function (argument) {
      var end = new Date() - start,
          hrend = process.hrtime(hrstart);
      console.info("Execution time: %dms", end);
      console.info("Execution time (hr): %ds %dms", hrend[0], hrend[1]/1000000);
  }, 1);
  
5. ## Create a download progress bar in nodejs. there are 5 steps in the progress bar, each taking 10 secs to complete. ##

  The first step to do that is to install the respective package
  
  npm install progress
  
  Below is the snippet of those.
  
  const ProgressBar = require('progress')
  
  const bar = new ProgressBar(':bar', { total: 5 })
  const timer = setInterval(() => {
    bar.tick()
    if (bar.complete) {
      clearInterval(timer)
    }
  }, 10)
  
6. ## How to automatically generated the next id in Mongo db using spring boot with an example ##

  The below is the snippet of code which addresses the above problem. Let;s say we are using SpringData + MongoDB
  
  **SequenceService.java** :
  
  import static org.springframework.data.mongodb.core.FindAndModifyOptions.options;
  import static org.springframework.data.mongodb.core.query.Criteria.where;
  import static org.springframework.data.mongodb.core.query.Query.query;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.data.mongodb.core.MongoOperations;
  import org.springframework.data.mongodb.core.query.Update;
  import org.springframework.stereotype.Service;
  
  import com.model.RandomSequencer;
  
  
  @Service
  public class NextSequenceService {
      @Autowired private MongoOperations mongo;
  
      public int getNextSequence(String seqName)
      {
        RandomSequencer counter = mongo.findAndModify(
          query(where("_id").is(seqName)),
          new Update().inc("seq",1),
          options().returnNew(true).upsert(true),
          RandomSequencer.class);
      return counter.getSeq();
      }
  }
  
  **RandomSequencer.java** :
  
  import org.springframework.data.annotation.Id;
  import org.springframework.data.mongodb.core.mapping.Document;
  
  @Document(collection = "randomSequencer")
  public class RandomSequencer {
      @Id
      private String id;
      private int seq;
  
  // getters and setters
  }
  
  The insertion would be as follows.
  
  Entity entity = new Entity();
  entity.setProperty(nextSequenceService.getNextSequence("randomSequencer"));
  /* Rest all values */
  
  entityRepository.save(entity);

  
7. ## Name any key management tool/framework to can manage the internationalization which can be integrated with the microservice ##
  
   In our current project we use Angular 6 as the frontend and we maintain the internationalization in English and Dutch
   based on the Locale. With that we use **ngx-translate** library It supports well with microservices developed in node/java.
   
   ngx-translate is an internationalization library for Angular which tries to close the gap between 
   the missing features of the built-in internationalization functionalities.
   
   The below are the respective npm modules required for the to implement.
   
   npm install @ngx-translate/core @ngx-translate/http-loader rxjs --save
   
   With rxjs,  Asynchronous call to respective microservices are made to retrieve the data from the backend(node/java) and based on the
   user language preference which is stored in the browsers local/session storage the translation keys in the json for the respective
   language are been transformed.
   
8. ## How can I convert a list into Binary in perl with an example ##
   
   **Perl pack Function**
   
   The pack function is used to convert a list into a string, according to a user-defined template (ex. a binary representation of a list). 
   
   The pack function has as arguments a LIST of values and a TEMPLATE. It concatenates into a string the list values converted according to 
   the formats specified by the template. It returns the resulting string.
   
   Its mainly purpose is to turn data (numbers and strings) into a sequence of bits that can be easily used by some external applications. 
   You can use the Perl pack function either to achieve binary data to a file or for network transmission.
   
   The reverse of this function is the unpack function which takes a sequence of bits and converts it into numbers and strings, needed for further processing.
   
   The syntax form of the pack function is as follows:
   
   STRING = pack TEMPLATE, LIST
   
   
  The TEMPLATE consists of a sequence of characters as shown in the table below. One or more modifiers may follow some letters in the template.
   
  The following is an example of that.
  
  #!/usr/bin/perl -w
  
  $bits = pack("c", 65);
  # prints A, which is ASCII 65.
  print "bits are $bits\n";
  $bits = pack( "x" );
  # $bits is now a null chracter.
  print "bits are $bits\n";
  $bits = pack( "sai", 255, "T", 30 );
  # creates a seven charcter string on most computers'
  print "bits are $bits\n";
  
  @array = unpack( "sai", "$bits" );
  
  #Array now contains three elements: 255, T and 30.
  print "Array $array[0]\n";
  print "Array $array[1]\n";
  print "Array $array[2]\n";
  
  Output :
  
  bits are A
  bits are 
  bits are �T
  Array 255
  Array T
  Array 30
  
9. ## How to include 2 html pages inside the html page using angularjs with an example ##
  
  AngularJS provides functionality to include another angularJS file in the current file. 
  By using **ng-include** directive.
  
  The purpose of ng-include are
     
     Fetch the external HTML fragments in our main angular application.
     To compile the external HTML fragments in our main angular application.
     To include the external HTML fragments in our main angular application.
    
  The below snippet code demonstrates the sample
    
  File1.html :
  
  <!DOCTYPE html>
  <html>
  <body>
  <h1>Include 1 Header</h1>
  <p>Using ng-include, this text 1 has been included into the HTML page!
  </p>
  </body>
  </html>
  
  File2.html :
    
 <!DOCTYPE html>
  <html>
  <body>
  <h1>Include 2 Header</h1>
  <p>Using ng-include, this text 2 has been included into the HTML page!
  </p>
  </body>
  </html>
  
  Main.html :
  
  <!DOCTYPE html>
  <html>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.9/angular.min.js<!DOCTYPE html>
  <html>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.9/angular.min.js"></script>
  <body ng-app="">
  <div ng-include="File1.html'"></div>
  <div ng-include="File2.html'"></div>
  </body>
  </html>
  s"></script>
  <body ng-app="">
  
 Therefore, we can conclude that include directive provides a way for code reusability. 
 Using ng-include directive, we can access one angular file in another angular file. 
 Without using ng-include in AngularJS we have to write the same code of one file again in another file to perform the same logic.
 
 Hence, we can say that ng- include directives are used for injecting the code of an external file in our main file so that, 
 we can reuse the logic present in the external file in the main file without re-writing it.