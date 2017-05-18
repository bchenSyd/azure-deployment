# deploy via node-cli

kudu debug  service:  https://{site name}.scm.azurewebsites.net/
* <strong style='color:red'> when deploying from local git, azure only deploy  `azure:master` branch</strong><br/>
* <strong style='color:green'> run `$git push azure adslot:master`  if your local branch isn't master</strong>

!["kudu"](./kudu_azure.png  "kudu")

## step 1: update your project to accomodate azure run time

* package.json: specify node engine to be at least v 5.5.0, otherwise you have problem with npm install ( and long path name mess on windows 2012)

```
  "engines": {
    "node": "5.5.0"
   }, 
```

* devServer, or express, must specify var port = process.env.port || 8000
>the reason behind is that node.js is behind iisnode, and you can't specify the port number iisnode uses

```
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config');

var port = process.env.port || 8000
new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  historyApiFallback: true
}).listen(port, 'localhost', function (err, result) {
  if (err) {
    return console.log(err);
  }

  console.log('Listening at http://localhost:' + port + '/');
});

```

* you can specify iisnode settings by add a iisnode.yml file at your project root

> below settings redirect stdout and stderr logs from iisnode into D:\home\site\wwwroot**iisnode** directory.

```
loggingEnabled: true
logDirectory: iisnode
```


# deployment

ask azure for help
```
$ azure help site list                                                                                                                                      
info:    Executing command help                                                                                                                             
help:    List your web sites                                                                                                                                
help:                                                                                                                                                       
help:    Usage: site list [options] [name]                                                                                                                  
help:                                                                                                                                                       
help:    Options:                                                                                                                                           
help:      -h, --help               output usage information                                                                                                
help:      -v, --verbose            use verbose output                                                                                                      
help:      -vv                      more verbose with debug output                                                                                          
help:      --json                   use json output                                                                                                         
help:      -s, --subscription <id>  the subscription id                                                                                                     
help:                                                                                                                                                       
help:    Current Mode: asm (Azure Service Management)         
```
install azure-cli globally
```
 $ npm install -g  azure-cli
 
 $ npm list -g azure-cli
   azure-cli@0.10.4 
```


 change azure mode to classic (asm), by default , azure run in resource manager mode (arm)

```
$ azure config mode asm                                                                                                                                     
info:    Executing command config mode                                                                                                                      
info:    New mode is asm                                                                                                                                    
info:    config mode command OK     
```

 login to azure (integrated login)
```
$ azure login                                                                                                                                               
info:    Executing command login                                                                                                                            
/info:    To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code HLT5WT882 to authenticate.
```

 create an azure site

```
$ azure site create --git bo-adslot                                                                                                                         
info:    Executing command site create                                                                                                                      
+ Getting sites                                                                                                                                             
+ Getting locations                                                                                                                                         
help:    Location:                                                                                                                                          
  1) South Central US                                                                                                                                       
  2) North Europe                                                                                                                                           
  3) West Europe                                                                                                                                            
  4) Southeast Asia                                                                                                                                         
  5) East Asia                                                                                                                                              
  6) West US                                                                                                                                                
  7) East US                                                                                                                                                
  8) Japan West                                                                                                                                             
  9) Japan East                                                                                                                                             
  10) East US 2                                                                                                                                             
  11) North Central US                                                                                                                                      
  12) Central US                                                                                                                                            
  13) Brazil South                                                                                                                                          
  14) Australia East                                                                                                                                        
  15) Australia Southeast                                                                                                                                   
  16) Canada Central                                                                                                                                        
  17) Canada East                                                                                                                                           
  18) West Central US                                                                                                                                       
  19) West US 2                                                                                                                                             
  20) UK West                                                                                                                                               
  21) UK South                                                                                                                                              
: 15                                                                                                                                                        
info:    Creating a new web site at bo-adslot.azurewebsites.net                                                                                             
\info:    Created website at bo-adslot.azurewebsites.net                                                                                                    
+                                                                                                                                                           
+ Getting locations                                                                                                                                         
info:    Initializing remote Azure repository                                                                                                               
+ Updating site information                                                                                                                                 
info:    Remote azure repository initialized                                                                                                                
+ Getting site information                                                                                                                                  
+ Getting user information                                                                                                                                  
info:    Executing `git remote add azure https://bambora@bo-adslot.scm.azurewebsites.net/bo-adslot.git`                                                     
info:    A new remote, 'azure', has been added to your local git repository                                                                                 
info:    Use git locally to make changes to your site, commit, and then use 'git push azure master' to deploy to Azure                                      
info:    site create command OK                
```

### NOTE
1. check portal for git-url. it may have port# in it. e.g. https://username@site-name.scm.azurewebsite.net:443/site-name.git
   if it's the case, run `git remote set-url azure url-with-port#
   after `git push azure master`, the windows authentication pops up (only in windows 10?), click Cancel and this will default to Basic authentication (type password in console)
2. delete C:\Users\{user name}\.azure directory, if you get a  'cannot find webspace ussouthcenter ' error
3. run `git remote -v` and you can see a azure remote respository is already added to your local respoitory
```
$ git remote -v                                                                                          
azure   https://bambora@quantas-api.scm.azurewebsites.net/quantas-api.git (fetch)
azure   https://bambora@quantas-api.scm.azurewebsites.net/quantas-api.git (push)
```
 check site status

```
$ azure site list                                                                                                                                           
info:    Executing command site list                                                                                                                        
+ Getting locations                                                                                                                                         
+ Getting sites                                                                                                                                             
data:    Name               Slot  Status   Location             SKU   URL                                                                                   
data:    -----------------  ----  -------  -------------------  ----  -----------------------------------                                                   
data:    bo-adslot                Running  Australia Southeast  Free  bo-adslot.azurewebsites.net                                                           
data:    redux-form-sample        Running  Australia Southeast  Free  redux-form-sample.azurewebsites.net                                                   
info:    site list command OK               
```

you can also generator deployment script and customize it futher, but this is not mandatory

```
#this will generate deployment script for you to do further customization. not mandatory though...
azure site deploymentscript --node
```



##  kudu service console:    https://bo-adslot.scm.azurewebsites.net/DebugConsole

 push local repository to azure -------- deployment username: bambora password: Password01

```
$ git push azure master   / $git push azure adslot:master                                                                                                                                   
Counting objects: 101, done.                                                                                                                                
Delta compression using up to 4 threads.                                                                                                                    
Compressing objects: 100% (93/93), done.                                                                                                                    
Writing objects: 100% (101/101), 137.16 KiB | 0 bytes/s, done.                                                                                              
Total 101 (delta 35), reused 0 (delta 0)                                                                                                                    
remote: Updating branch 'master'.                                                                                                                           
remote: Updating submodules.                                                                                                                                
remote: Preparing deployment for commit id 'a2f2eaa1d9'.                                                                                                    
remote: Generating deployment script.                                                                                                                       
remote: Generating deployment script for node.js Web Site                                                                                                   
remote: Generated deployment script files                                                                                                                   
remote: Running deployment command...                                                                                                                       
remote: Handling node.js deployment.                                                                                                                        
remote: KuduSync.NET from: 'D:\home\site\repository' to: 'D:\home\site\wwwroot'                                                                             
remote: Deleting file: 'hostingstart.html'                                                                                                                  
remote: Copying file: '.babelrc'                                                                                                                            
remote: Copying file: '.eslintrc'                                                                                                                           
remote: Copying file: '.gitignore'                                                                                                                          
remote: Copying file: 'devServer.js'                                                                                                                        
remote: Copying file: 'favicon.ico'                                                                                                                         
remote: Copying file: 'index.html'                                                                                                                          
remote: Copying file: 'package.json'                                                                                                                        
remote: Copying file: 'README.md'                                                                                                                           
remote: Copying file: 'webpack.config.js'                                                                                                                   
remote: Copying file: 'src\index.js'                                                                                                                        
remote: Copying file: 'src\rootReducer.js'                                                                                                                  
remote: Copying file: 'src\api\payload.json'                                                                                                                
remote: Copying file: 'src\components\asyncValidate.js'                                                                                                     
remote: Copying file: 'src\components\index.js'                                                                                                             
remote: Copying file: 'src\components\MaterialUiForm.js'                                                                                                    
remote: Copying file: 'src\store\configureStore.js'                                                                                                         
remote: Copying file: 'src\style\base.scss'                                                                                                                 
remote: Copying file: 'src\style\logo.PNG'                                                                                                                  
remote: Copying file: 'src\style\_index.scss'                                                                                                               
remote: Using start-up script devServer.js from package.json.                                                                                               
remote: Generated web.config.                                                                                                                               
remote: The package.json file does not specify node.js engine version constraints.                                                                          
remote: The node.js application will run with the default node.js version 4.4.7.                                                                            
remote: Selected npm version 2.15.8                                                                                                                         
remote: ...... 
```

to get deployment status run 

```
D:\home\site\deployments\b2a1bd33b68f669492532441fdc1ba6be172a136>cat status.xml
```

kudu service \ debug console \  D:\home\site\wwwroot\iisnode>  tail -f 1b378f-26760-stdout-1475551526465.txt
