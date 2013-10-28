##Parthenon: Threat Intelligence Sharing Using Splunk

###Introduction:

Threat intelligence sharing is a difficult task for organizations. Information is hard to compile and maintain and its difficult to disseminate or acquire information in a timely fashion. We demonstrate an intelligence sharing solution; Parthenon. It is completely contained within the Splunk platform. Parthenon allows for intelligence sharing and retrieval through Splunk's REST API. All access controls are implemented using Splunk's access model. And since Splunk is a robust, big data platform, Parthenon data can be stored and analyzed indefinitely. Parthenon includes visualizations and automated alerts for actionable intelligence.

###Description:

Analysts acquire threat intelligence in the course of incident response and forensics activities. But due to the cumbersome nature of intelligence sharing, much information goes unshared and unreported. We wanted to have the simplest possible mechanism so maintenance, sharing and retrieval would be as painless as possible.Using SSL over HTTP as the mechanism for all submissions and acquisitions, analysts can use anything from a web browser to home-grown applications for interactions.

Submissions are made using the Splunk REST API with a predefined syntax. The submitted information is parsed and indexed by Splunk (Section 3 describes the submission process in more detail). For retrieval, a requester makes a REST API request via HTTP using a predefined syntax. Results for the request can be acquired in a variety of formats (Section 4 describes the retrieval process in detail). 

Status dashboards and Visualizations allow Parthenon administrators and contributors to keep track of submissions and acquisitions. The dashboards also provide insight into additional information and third party enrichments. Enrichments are also used to identify high profile data e.g. domains that might be on the Alexa top 1000 list. In addition, participants can subscribe to real-time, automated updates. E.g. if Alice adds a malicious domain to Parthenon, Bob can be automatically notified of the update and the contents of the update. (Section 5 describes visualizations, enrichments and alerts)

Because Parthenon is designed to be a multi-user/multi-party collaboration system, there is some setup required to create user accounts with appropriate permissions. Details of this setup are discussed in section 2.

###Parthenon Installation and Setup:

Parthenon is a Splunk app. It can be installed just like any other Splunk app; either via the Web UI or via the command line. 

###Installation via the Web UI: 

Navigate to Manager >> Apps >> Find More Apps Online. Search for Parthenon in the searh bar (top right). Click on Install Free. 

###Installation via the command line:

Download the Parthenon package from SplunkBase; www.splunkbase.com . Rename the parthenon.spl file to parthenon.tgz. Unpack the tar archive. On a linux system, you can type: tar -vxzf parthenon.tgz . On a windows system, you can download something like 7-zip to unpack the tar archive.

Once unpacked, move the entire directory, parthenon, into $SPLUNK_HOME/etc/apps/ . 
Restart Splunk: $SPLUNK_HOME/bin/splunk restart

###Creating a new role:

Once the App is installed. Create a role for all users of Parthenon. From the Web UI, go to Manager > Access Controls > Roles > New
Add New
    Role Name : Parthenon
    Default App: Parthenon
    Search Restrictions
    Restrict search terms: index=parthenon
    Restrict search time range: 86400 , or one day
    Limit concurrent search jobs: 3
    Limit concurrent real-time search jobs: 0
    Limit total jobs disk quota: <leave blank>
    Inheritance
    Available Roles: admin
    Capabilities: edit_tcp, output_file, search
    Indexes searched by default
    Selected indexes: parthenon
    Indexes
    Selected search indexes: parthenon

Click on Save. 

###Creating User Accounts

Create individual accounts for the collaborators to submit threat data and retrieve threat information. From the Web UI go to:
Manager > Access Controls > Users > New 
Add new
Username: <the user name of the new user>
Full Name: <Name of the user>
Email Address: <Email of the user>
Time Zone: Default System Timezone
Default app: <Leave Blank>
Assign to role
Selected roles: parthenon
Set Password
Password: <the users password> 
Confirm password

###Submitting Data

Threat data is submitted via a POST request. The request must contain an actor field or a md5sum field and optionally, any number of optional fields e.g. a comment field. Fields and values are represented as key=value pairs. The actor field can be a domain name or an IP address. Since submissions are made via an HTTP post, they can be made via curl or some python script.

Example 1:

    curl -k -u monzy:monzypass -d "actor=www.monzyerza.com comment='This domain has been scanning for quite some time. Block immediately'" "https://localhost:8089/services/receivers/simple?index=parthenon&sourcetype=threat"

In this example, we submit the actor www.monzymerza with a comment. Notice that the comment is surrounded by a single quote.

Example 2: 

    curl -k -u monzy:monzypass -d "md5sum=1756f939a1a623c8547daf0c8ac897fa comment='second stage dropper' sourceurl=reallybadsite.com" "https://localhost:8089/services/receivers/simple?index=parthenon&sourcetype=threat"

In this example, we submit an md5sum and an accompanying comment. but we also include a sourceurl field to specify the source url for the malware. 

###Retrieving the Threat List

The threats are retrieved by way of a Splunk search. The output can be received in XML, JSON or cvs. 

In this linux example, we download all the threat intel that was uploaded today. 

Submit a search and get the search results:
     `curl -s -u part:part -k 'https://localhost:8089/services/search/jobs' --data-urlencode 'search=search index=parthenon earliest=@d' -d exec_mode=oneshot -d output_mode=csv`

###Dasboards and Alerts
Finish me

Appendix A: Splunk REST API 
API Basics: http://docs.splunk.com/Documentation/Splunk/latest/RESTAPI/RESTusing
Search and retrieval: http://docs.splunk.com/Documentation/Splunk/5.0.2/RESTAPI/RESTsearches

TODO: 
Create a script to create the Parthenon role

