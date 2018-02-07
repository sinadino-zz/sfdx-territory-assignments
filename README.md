# Install DX Travis CI in 6 Steps 

Installing travis ci and necessary steps to run ci jobs. This assumes you have a github account and your dx project github repository is connected to Travis already. After you go through these 6 steps, Travis CI will be invoked every time you push changes to your github repo. 

## Getting Started

```
cd into your DX project folder.
copy or clone travis.yml file 
create an “assets” folder ( this is where your ssl certificates will go)
create your git project and connect to your github repo(this should be the same Travis is connected to).
```

### Prerequisites

Install Travis
Make sure you have at least ruby 1.9.3 then install travis
```
$ruby -v
```
```
$gem install travis -v 1.8.8 --no-rdoc --no-ri
```
```
$travis version
```


### Create SSl Certificate 

The files created during this step contain sensitive information that others can use to compromise your system. So it’s important to keep track of them in a safe place to use later. To make it easier to locate these files, create a folder in your file system that is outside of the project folder.

```
Create a folder certificates and cd into it ( or if folder exists then delete the content of it).
```
```
From the certificates folder run
$openssl genrsa -des3 -passout pass:x -out server.pass.key 2048

```
```
Then create a key file
$openssl rsa -passin pass:x -in server.pass.key -out server.key
```
```
Delete the server.pass.key
$rm server.pass.key
```
```
Request the certificate
$openssl req -new -key server.key -out server.csr
```
```
Enter all requested info. Enter "." if you want to skip a prompt/question
```
```
Generate the SSl certificate
$openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt
```
The server.crt file is your site certificate, suitable for use with the connected app along with the server.key private key


###  Create connected app in Salesforce
- From Setup, enter App Manager in the Quick Find box, then select App Manager.
- Click New Connected App.
- Enter the connected app name and your email address:
  * Connected App Name: sfdx travis ci
  * Contact Email: <your email address>
- Select Enable OAuth Settings.
- Enter the callback URL: http://localhost:1717/OauthRedirect
- Select Use digital signatures.
- To upload your server.crt file, click Choose File.
- For OAuth scopes, add:
  * Access and manage your data (api)
  * Perform requests on your behalf at any time (refresh_token, offline_access)
  * Provide access to your data via the Web (web)

After you’ve saved your connected app, edit the policies to enable the connected app to circumvent the manual login process.

- Click Manage.
- Click Edit Policies.
- In the OAuth policies section, for Permitted Users select Admin approved users are pre-authorized, then   click OK.
- Click Save.

### Create a Permission Set

Create a permission set and assign pre-authorized users for this connected app.

- From Setup, enter Permission in the Quick Find box, then select Permission Sets.
- Click New.
- For the Label, enter: sfdx travis ci
- Click Save.
- Click sfdx travis ci | Manage Assignments | Add Assignments.
- Select the checkbox next to your Dev Hub username, then click Assign | Done.
- Go back to your connected app.
  * From Setup, enter App Manager in the Quick Find box, then select App Manager.
  * Next to sfdx travis ci, click the list item drop-down arrow (), then select Manage.
  * In the Permission Sets section, click Manage Permission Sets.
  * Select the checkbox next to sfdx travis ci, then click Save.


### Test the JWT Auth Flow

$export CONSUMER_KEY=<connected app consumer key>
$export JWT_KEY_FILE=<your server.key path>
$export HUB_USERNAME=<your Dev Hub username>

Then run:
$sfdx force:auth:jwt:grant --clientid ${CONSUMER_KEY} --username ${HUB_USERNAME}  --jwtkeyfile ${JWT_KEY_FILE} --setdefaultdevhubusername




###  Encrypt Your Secrets
    For Travis CI to successfully execute the JWT bearer flow on your behalf, it requires access to the server.key so it can sign the OAuth request. To perform this step securely, encrypt the server.key so that only Travis CI can decrypt it, and then add it to your project.




```
 From a command window, change to your local working sfdx  project directory.
```
```
 In the assets folder, delete anything in it every time you start a new dx project( .the sample server.key.enc file).
```
```
 Copy your server.key from the certificates directory to the assets folder.

```
```
 Log in to Travis CI with your GitHub credentials:
 $travis login --org
```
```
 $travis encrypt-file assets/server.key assets/server.key.enc --add
```

Using the Travis CI CLI, run the following commands:
$travis env set CONSUMERKEY <connected app consumer key>
$travis env set USERNAME <your Dev Hub username>




## Done

You are ready to try pushing your changes to your github repo then go to Travis and wait for testing results.


## references

* [SFDC DX Tools](https://github.com/amphro/salesforcedx-tools) - The Salesforce DX tools 
* [Trailhead](https://trailhead.salesforce.com/en/modules/sfdx_travis_ci/units/sfdx_travis_ci_setup) - Travis CI Trailhead
*  [Travis](https://github.com/travis-ci/travis.rb#installation) - Installing Travis
*  [Travis Site](https://travis-ci.org/) - Travis Website

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Cristiano Sinadino** - *Initial work* - [PurpleBooth](https://github.com/sinadino)



## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
