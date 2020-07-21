---
title: "Jira APIs in a Python app"
date: 2020-07-17
slug: "blog1"
description: "How to build an web app using Jira APIs in Python"
keywords: ["blog", "tech-blog", "jira", "python"]
draft: false
tags: ["blog", "tech-blog", "jira", "python"]
math: false
toc: false
---

{{< figure src="/blog/images/python-logo.png" caption="" >}}

Imagine your filling your Jira tasks everyday and by the end of the week you wonder, can I find the total number of hours logged in Jira.

So what do you do? As any sane(insane) person you think of developing an webapp to fetch all the data and calculate your day wise work logged for each story/task/subtask or SR. Then you start searching on how to build an webapp using Jira APIs. 

Okay , disclamaier, there is already a way to get this report in Jira, go to Reports→ Time report and voila! you get the report. But, whats the fun in that. So if you are like me and want to avoid do this the easy way, here are the steps to do the hard way.

- Step 1: Go the jira API page

Find the jira API in 

[JIRA 7.6.1](https://docs.atlassian.com/software/jira/docs/api/REST/7.6.1/#api/2/issue-getIssueWorklog)

- Step 2: Choose the language and start coding

I wanted a quick and easy language to work with, so I chose python. You can chose any language of your choice.

Step 2.1 :

First get the jira cookie.

```python
import requests
import json
import datetime

def login(loginUserName, loginPassword):
    loginAPI = 'auth/1/session'
    loginURL = baseURL + loginAPI #baseURL is the jira URL. This can be a company specific Jira URL
    loginData = {"username": loginUserName, "password": loginPassword}

    loginResponse = requests.post(loginURL, json=loginData)

    if loginResponse.status_code != 200:
        raise Exception('POST ' + loginURL + ' {}'.format(loginResponse.status_code))
    else:
        myJessionID = loginResponse.cookies['JSESSIONID']

return myJessionID
```

Step 2.2:

Once you get the cookie, call the Apis to get all the issues which you have updated for the current week. 

```python
#Below method gets the issues updated by a user in the current week
def issuePicker(sessionID , username):
    issuePickerAPI = 'api/2/search?jql=updatedDate%3E%3DstartOfWeek()%20AND%20worklogAuthor%3D'+username+'&fields=issues,summary'
    url = baseURL + issuePickerAPI
    print(url)
    myCookie = dict(JSESSIONID=sessionID)
    resp = requests.get(url, cookies=myCookie)

    if resp.status_code != 200:
        raise Exception('GET ' + url + ' {}'.format(resp.status_code))
    else:
        #print("resp: {}".format(resp.json()))
        json_decode = json.loads(json.dumps(resp.json()['issues']))
        issueList=[]
        for item in json_decode:
            issueDict = {}
            issueDict['issueName'] = item.get('key')
            issueDict['issueId'] = item.get('id')
            fieldsDict = item.get('fields')
            issueDict['summary'] = fieldsDict.get('summary')
            issueList.append(issueDict)
        return issueList
```

Step 2.3

Once you get the worklogs that are updated, you need to get the time spent in these worklogs. Finally to divide this data, daywise you need do to some python magic. You can do this using below code

```python
#Below method gets the worklogs associated with the issue. Also the below method addes the worklogs hours and splits it according to day and issue.
def issueWorklog(sessionID, issueId , user):
    issueWorkLogAPI = 'api/2/issue/'+'{}'.format(issueId)+'/worklog'
    url = baseURL + issueWorkLogAPI
    print(url)
    myCookie = dict(JSESSIONID=sessionID)
    resp = requests.get(url, cookies=myCookie)

    if resp.status_code != 200:
        raise Exception('GET ' + url + ' {}'.format(resp.status_code))
    else:
        print("resp: {}".format(resp.json()))
        json_decode = json.loads(json.dumps(resp.json()['worklogs']))
        worklog_list = []
        for item in json_decode:
            startOfWeek = datetime.datetime.today() - datetime.timedelta(days=datetime.datetime.today().isoweekday() % 7)#This gets time Sunday midnight
            startOfWeek = startOfWeek.replace(hour=00,minute=00,second=0,microsecond=0)
            updatedTimeSplitArray = (item.get('started')).split('T')
            updatedTime = datetime.datetime.strptime(updatedTimeSplitArray[0],"%Y-%m-%d") 
            if((item.get('updateAuthor').get('key') == user) and (updatedTime >= startOfWeek)):#get all the worklogs updated by the user and updated after 1 week from current date
                worklog_dict = {}
                worklog_dict['user'] = item.get('updateAuthor').get('key')
                worklog_dict['day']=datetime.datetime.strptime(updatedTimeSplitArray[0], "%Y-%m-%d").weekday() 
                worklog_dict['updated'] = item.get('started')
                worklog_dict['timeSpent'] = item.get('timeSpent')
                worklog_dict['timeSpentSeconds'] = item.get('timeSpentSeconds')
                worklog_dict['comment'] = item.get('comment')
                print(worklog_dict)
                worklog_list.append(worklog_dict)
        print(worklog_list)
    return worklog_list
```

Step 3 : Display the data

Since I chose Python, it was only natural I would chose a Python web framework. Python flask was a obvious choice, it being lightweight and all. You can opt for Django, but flask fitted my needs well.

```python
from flask import Flask , render_template , request
import datetime

app = Flask(__name__)
global jiraSessionId

@app.route('/worklog' , methods=["POST"])
def jworklog():
    username = request.form.get("username")
    password = request.form.get("password")
    if not username or not password:
        return render_template("index.html" , errorMessage = "Username or password cannot be empty")    
    else:
        try:
            jiraSessionId = login(username, password)
            completeIssueDetailsList = newGetWorkLogs(jiraSessionId , username)
            worklogDict = completeIssueDetailsList[0]
            summaryDict = completeIssueDetailsList[1]
            return render_template("worklog.html" , worklogDict = convertDictFromSecondsToHours(worklogDict) , summaryDict = summaryDict ,  username = username ) 
        except:
            return render_template("index.html" , errorMessage = "Something went wrong")
```

And Voila!, the your jira app should be up and running.

