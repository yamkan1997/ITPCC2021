# collector-coin
NYU - ITP Collector Coin project

Developed during the summer semester 2021

### To Run
download the project repo.

from the project root run: `docker-compose up`

contents of the .env in the Web folder:
```
REACT_APP_STABLE_TOKEN_ADDRESS=0x0c118276ba64fa8D6Af7b3B74a1AE8F277E0BF25
REACT_APP_FAUCET_ADDRESS=0x23c2917452f5eC19F5CFF430C2FB221DcdA9a076
REACT_APP_CC_TOKEN_ADDRESS=0x08Fde9a70e446E1eDE3667F044AA55A2D3a53043
REACT_APP_FACTORY_ADDRESS=0x0ac94Dbac97E3ae4fa6a6b02785e07a6d18ab6F9
REACT_APP_API_DEPLOY_URL=http://localhost:5000
REACT_APP_API_JBPM_URL=http://localhost:8080
REACT_APP_API_ETHERSCAN_URL=https://rinkeby.etherscan.io/address

```
** Note:

`REACT_APP_API_DEPLOY_URL=http://{server-name}:5000`

`REACT_APP_API_JBPM_URL=http://{server-name}:8080`

replace the server names with the machine id that is hosting the containers.

live demo was staged at: 208.113.132.193, hence:

`REACT_APP_API_DEPLOY_URL=http://208.113.132.193:5000`

`REACT_APP_API_JBPM_URL=http://208.113.132.193:8080`

jBPM is available from: `localhost:8080/business-center`

frontend is available at: `localhost:3000`

REST documentation at: `localhost:8080/kie-server/docs`

REST commands need Basic authentication to kie-server:

`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/conainters/test-live_1.0.0-SNAPSHOT/processes/test-live.hello/instances`


Login to postgres server:
login to machine: `docker-compose exec postgres bash`
login to db: `psql -d jbpm -U jbpm`
list tables: `\d`
quit psql: `\q`

### REST commands

Process information:
`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/containers/Submission_Process_1.0.0-SNAPSHOT`

Start Process NewListing: POST
`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/containers/Submission_Process_1.0.0-SNAPSHOT/processes/Sumbission_Process.NewListing/instances`

include variables in body: 
```
{
    "id": "short-uuid",
    "vin": "12345",
    "make": "Car",
    "model": "Fast",
    "vinMatched": true,
    "msrp": "2000",
    "fundingGoal": "5000"
}
```

returns process instance id

Query Tasks for process id 10: GET
`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/queries/tasks/instances/process/10`

Start Task 14: PUT
`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/containers/Submission_Process_1.0.0-SNAPSHOT/tasks/14/states/started`

Query variable fields from listing:
`http://localhost:8080/kie-server/services/rest/server/queries/processes/instances/55/variables/instances`

Complete Task 14: PUT
`http://wbadmin:wbadmin@localhost:8080/kie-server/services/rest/server/containers/Submission_Process_1.0.0-SNAPSHOT/tasks/14/states/completed`


### Contract Operations
After listing approval, jBPM fires REST request
to deploy a new project. The following are steps 
to interact with that project:

* Funder approves transfer from Stable Token contract
to CCToken contract

* Funder swaps Stable Tokens to CCTokens
  
* Funder approves transfer from CCToken contract to
new project contract

* Funder transfers funds to new project contract, registering a share in proceeds
  
* CC admin deactivates funding on project
  
* CC admin withdraws funding from project to CCFactory

* CC admin transfers funding to restorer

* Once car is sold, CC admin transfers proceeds to CCFactory
  
* CC admin approves transfer from CCFactory to project
  
* CC admin transfers proceeds from CCFactory to project

* Funder can now recoup her share of proceeds from the project

### Starting the project

run `docker-compose up` to create and run the containers

navigate to `localhost:8080/business-central` and log in
with user: `wbadmin` abd pass: `wbadmin`

If NOT using toby's image, the following steps
need to be taken to set up the BPM process::>

create a space

create a new project called `Submission_Process`

Import the `NewListing` asset from the bpmn folder into the 
com.myspace.submission_process package, name the asset `NewListing`

Register the REST work item handler, by navigating to `Work Definitions`
in the project, and copying the REST service `defaultHandler` field:
`new org.jbpm.process.workitem.rest.RESTWorkItemHandler()`

In the same project navigate to `Settings->Deployments->Work Item Handlers`
and add a new Work Item Handler, name: `Rest` and value:
`new org.jbpm.process.workitem.rest.RESTWorkItemHandler()`

Save the current settings

Deploy the process

The jBPM is now ready to receive commands for this process

### Restarting Wildfly
If the wildfly server needs to be restarted

(to add the CORS filters manually)

run the following command from `Wildfly/bin`:

`./jboss-cli.sh --connet command=:reload`
