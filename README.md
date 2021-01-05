# splunk-concepts


source="users-202012301346-1.csv" host="kubemaster" sourcetype="csv" | table phone_no status full_name



example active/deactive users
source="users-202012301346-1.csv" host="kubemaster" sourcetype="csv" | stats count by status

you can do visualization diffrent charts available

splunk archetecture devided into 2

processing component---> forwoder , indexers, serach head
managment component  ----> 
                          monitoring console, 
                          index cluster master(data replucation on indexs),
                          deployment server (centralize configuration manager), 
                          search head cluster deployer, 
                          licence master

indexers cluster called peer nodes


licenec type ---> import how much data, 24hr time period how to clacluate , per day

standard --> u need to purchase a licence example 5gb/day avialble in custom

entrprise trial 500mb/day

dev/test licence are restricted to non production env

free licence ----> 500mb/days , retention period 60 days

industrial IOT Licence

Forworder licence


licence violation

1 day daily volume exceed limit ----> warning

if 5 such warning /in 30 days ----> violation 

once u are in violation splunk send continious messages


universal forwoder does not require licence where is heavy forwoder just require forworder licence


licence group ---licence stack (contatines actaul licence files) ---licence pool ---assign to instince  ---> this whole process called distributed licencing

note: mutiple component need licence so licence manger keep it centerally so all can consume it


splunk conf files:

input.conf --> for input data
propes.conf --> for indexers
servers.conf ---> cluster setup
transform.conf---> setting value that control data transformation

configuratiin folder---> splunk_home ---> etc/user etc/app etc/system  there will be folder called default and local





any configuration file override use local folder to change not default
 precedency of config files
 
Global Context
 -------------------------------
 system local
   app local
     app default
       system default   
       
SAAS
       
App Context/User Context
--------------------------------
user dircetory for current user
app local 
app default
system local
system default

btool to examin configuration settings

./splunk cmd btool app list

./splunk cmd btool transforms list --app=aqua_security

./splunk cmd btool transforms list --app=search --debug

./splunk cmd btool transforms check ------> for typo


indexes contains 3 type of data ---> compressed data, raw data, metadata

stanza-- header in [] example [id]

Index Type:
--------------------
1 . event data -- can handle any data type
2.  metrics data -- can store and retrive matrix data


matrix data contains 4 things ---matric name, timestamp, dimesion ,value



splunk pipeline


process anykind of data ----> trnasform into events---->indexes are store into buckets---> directory on the file system organizsed by age

rowdata ---- transform events ----> data+ metadata+timestamp+host+source+ourcetype

index path splunk_home/var/lib/splunk/*

indexes are 3 types

main
_internal
_audit

Type Of buckets:

Hot buckets
warm buckets
cold frozon
frozon  The frozen bucket is where archived data is stored.
thawed  In this bucket is archived data restored


root@kubemaster:/opt/splunk/var/lib/splunk/defaultdb# ll
total 24
drwx------  6 root root 4096 Jan  3 08:09 ./
drwx------ 16 root root 4096 Jan  3 09:20 ../
drwx------  2 root root 4096 Jan  3 08:39 colddb/
drwx------  2 root root 4096 Jan  3 08:09 datamodel_summary/
drwx------ 11 root root 4096 Jan  3 09:20 db/
drwx------  2 root root 4096 Jan  3 08:09 thaweddb/


splunk

source ----> index ------hot bucket(1st hash)  ----> warm path (2nd hash)

What are the benefits of Hashing? One main use of hashing is to compare two files for equality. Without opening two document files to compare them word-for-word, the calculated hash values of these files will allow the owner to know immediately if they are different.


fish bucket: keep track what files or part of files got indexed, helpful to not duplicate data using seek pointers use crc value to chec exits or not

by deafult data retention 6 years


example of .conf index
[main]
homePath   = $SPLUNK_DB/defaultdb/db
coldPath   = $SPLUNK_DB/defaultdb/colddb
thawedPath = $SPLUNK_DB/defaultdb/thaweddb
tstatsHomePath = volume:_splunk_summaries/defaultdb/datamodel_summary
maxMemMB = 20
maxConcurrentOptimizes = 6
maxHotIdleSecs = 86400
maxHotBuckets = 10
maxDataSize = auto_high_volume


In splunk users dones not assigned direct capabilities

but uses role which is collection of capabilities

type of role:

1. admin  -- can do anything
2. power --- can shared all objects, alerts,tag events etcs
3. user   --- create its own searches, modify run create and edit event type
4. can_delete -- use need to assign this to delete indexes 

roles can be added/delete in splunk web or authorize.conf

The three types of Splunk authentication are

native ldap scripted

mutifactor authentixation using duo


Data input type to splunk

input ----> parsing queue ----> parse ---indexing queue ----indexing ---search


during parsing phase splunk analysis and examin transform data by breaking data into events and add host,sourcetype,source,timestamp


type of inputs
1. file and directory input ----> here we add monitor to files and directory ---> files can be compressed but it uncompressed when procceds
    monitorNoHandler -----> available only for windows

2. network inputs   ---> data from udc/tcp snmp events syslogs
3. windows input ---> window event logs, registery ,AD,WMI
4  matrix data ---> 
5. fifo queue
6. scripted input 
7. modular input
8. http event controller


configure data inputs

using app
ui
cli splunk add monitor pathforfileordir
add stanza in input.conf


type of forwoders

universal forwoder: reacive data and sent to listen , reciver can be another forwoder system, reciver can be another forwoder system
                    it has no licence
                    simple take data from source and forword data
                    seprate instalations

--------------------------------------------------------------------------------------------------------
configuring setup universal forwoder in linux

on Splunk indexer node: 
 settings ----> click on forwarding and receving ---> Configure receiving --> new receiving port ---default port 9997


on forwoder node kubenode01---

download universal forwader tar ball
  ./splunk start
 sudo ./splunk enable boot-start                   

./splunk add forward-server 192.168.56.3:9997 

you can remove as well sudo ./splunk remove forward-server 192.168.56.3:9997
Splunk username: admin
Password: 
Added forwarding to: 192.168.56.3:9997.

./splunk add monitor /var/log
./splunk restart

you can see entries in /opt/splunkforwoder/etc/system/local/outputs.conf
------------------------------------------------------------------------------------------------------------
configure heavy forwader require same above setps (need full enterpise splunk installation) + licence

Splunk in HA and distributed system:
 DS = HA+PERFORMANCE+DISASTER

search head ----convert into ---search head cluster ----> there will be one captain if that goes done search head cluster will elect new 
indexers ----> convert into ---- indexer cluster
forwoder ---> load balance forwader

so deployer manage common configuration for search head 
so master cluster nodes make sure data replication acrosss index cluster


setup distribued system:

initilize clsuer member 

select captain

then we can add member


steps:

deployer goto splunk_home/etc/system/local/server.conf add cluster properties
ssh into member1 and splunk_home/bin 
 run ./splunk init sh-cluster-config
 restart splunk
ssh into member2 and splunk_home/bin 
 run ./splunk init sh-cluster-config
 restart splunk 

then elect captain:
./splunk bootstrap shcluster-captain 


/opt/splunk/etc/apps here all app will be installed

Configure deployment server app and clinet:
-----------------------------------------------------------------------------S
deployment server --- usually in search head server
deployment app ----> create in deployment server /opt/splunk/etc/deployment-app  ----> mkdir test

deployment clinets are nothing but forwader server

cd /opt/splunk/bin
and run sudo ./splunk reload deploy-server to reload deployment app


goto forwoder managment in setting and see test app will be list

create a server class and add that app to it

vagrant@kubenode01:~/splunkforwarder/bin$ sudo ./splunk set deploy-poll 192.168.56.2:8089
Your session is invalid.  Please login.
Splunk username: admin
Password: 
Configuration updated.'

vagrant@kubenode01:~/splunkforwarder/bin$ sudo ./splunk restart


goto forwoder managment in setting and see client  will be list
as well goto 
~/splunkforwarder/etc/app

drwx------  4 root    root    4096 Jan  4 19:04 test/

Which Splunk Enterprise component allows you to group and configure other Splunk components by common characteristics? ---> deployment-server

deployment server is for forwoders managemnet and segrigate them using app and server class only

minitor input manly 2 types:

1. monitor ----> continious monitor local/remote file/dir tcp/udc
2. upload  -----> one time
3. monitorNoHandler

monitor input will go into inputs.conf file

scripted input --> starnge log formate can be make undersatable by splunk using secripted input 
so basically its prepare data before splunk ingest it


tcp/udp does not require forwader

SNMP means scripted data

HEC http event collector monitor help to sned data over http basically helpful to send clinet side data example user actions 

clinet side configure token and hec endpoint so it can send dato splunk to index


how to set HEC:
--------------------------------------------------------------------
Forwoder not required
create http event collector token and port,sourcetype in global settings
use command

endpoint: http://splunk_indexer:8088/services/controll/raw/
header: Authorizatiob splunk 10278jagdjxxxxxxxtoken
body: "source,text,trial"

input pahse: 64k data splunk input data in utf-8 form that can change in prop.conf
parsing phase: break down into events called event line breaking ---> then set time stamp on each events ---> apply host,source,sourcetype----> trnasform if any


transforms.conf add regex so it search for data in input and mask it


inputs.conf
-------------
[monitor://var/logs]
sourcetye = syslogs

transforms.conf
---------------
[transformer-1]
REGEX: <regex>
FORMATE: <formate of mask>

props.conf
----------------
[source type from input.conf]
Trnasformer-anonamize = transformer-1


you can have SEDDMD regex to find and replace in props.conf
example creditcard number


/opt/splunk/etc/system/local create inputs.conf


Data mask:
----------------------------------------------
inputs.conf

[monitor:///tmp/ccdata.csv]
sourcetype = ccdata


props.conf

[ccdata]
SEDCMD-ccdatamask = s/\d+$/xxxxxxxxxxxxx/g


cat /tmp/ccdata.csv
    amar1,khan,bangalore,amarkotasky1@gmail.com,8310325441,4444555566667771
    amar2,khan,bangalore,amarkotasky2@gmail.com,8310325442,4444555566667772
    amar3,khan,bangalore,amarkotasky3@gmail.com,8310325443,4444555566667773
    amar4,khan,bangalore,amarkotasky4@gmail.com,8310325444,4444555566667774
    amar5,khan,bangalore,amarkotasky5@gmail.com,8310325445,4444555566667775
    amar6,khan,bangalore,amarkotasky6@gmail.com,8310325446,4444555566667776
    amar7,khan,bangalore,amarkotasky7@gmail.com,8310325447,4444555566667777
    amar8,khan,bangalore,amarkotasky8@gmail.com,8310325448,4444555566667778
    amar9,khan,bangalore,amarkotasky9@gmail.com,8310325449,4444555566667779


 you can add extract field to map data   
