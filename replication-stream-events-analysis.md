## Context

The purpose of this analysis is to try to understand the different events published by Gerrit instances in the different topologies.

There are **5 scenarios**:
- Scenario 1 -  primary1/primary2 stable-3.5 with pull-replication and maxApiPayloadSize=40000
- Scenario 2 -  primary1/primary2 stable-3.5 with pull-replication and maxApiPayloadSize=1
- Scenario 3 -  primary1/primary2 stable-3.5 with pull-replication, maxApiPayloadSize=40000 and events-kafka
- Scenario 4 -  primary1/primary2 stable-3.5 with pull-replication, maxApiPayloadSize=1 and events-kafka
- Scenario 5 -  primary1/primary2 stable-3.5 with push replication

And the **actions** taken in each scenario are the following:
- Create project project1
- Create project project2
- Delete project project2
- Create branch branch1 in project1
- Create a change in project1
- Clone project project1
- Checkout change
- Add a file and push to change
- Stop primary2
- Create project project3
- Create branch branch1 in project3
- Restart primary2
- ssh command to start pull-replication in primary2 with options: --all --wait

Notes: All the UI actions and git commands are executed against primary1.

### Scenario 1 -  primary1/primary2 stable-3.5 with pull-replication and maxApiPayloadSize=40000

**Create project repo1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"53a6611c31065fe215a4ff4eb69991e2ac723fe6","refName":"refs/meta/config","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686214378,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"53f8829b05bdc300877ee1078da57f686132806d","refName":"refs/heads/master","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686214378,"instanceId":"primary"}
"type":"project-created" =>     {"projectName":"repo1","headName":"refs/heads/master","type":"project-created","eventCreatedOn":1686214378,"instanceId":"primary"}

```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"repo1","ref":"..all..","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686214378,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"repo1","ref":"refs/meta/config","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686214378,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"repo1","ref":"refs/heads/master","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686214378,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"repo1","ref":"refs/meta/config","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686214379,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"repo1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686214379,"instanceId":"replica-1"}
```

**delete project repo2**
no events

**Create branch branch1 in project1**

- Events in primary1:
```
"type":"ref-updated" =>     {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"53f8829b05bdc300877ee1078da57f686132806d","refName":"refs/heads/branch1","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686214571,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"repo1","ref":"refs/heads/branch1","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686214571,"instanceId":"replica-1"}
```

**Create a change in project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"b9f6a527c7cb91c44aad808f9bf1ff3097d3596b","refName":"refs/changes/01/1/1","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686214672,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"1796cd0797c26285cc9a7fb681582a9f10813fd5","refName":"refs/changes/01/1/meta","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686214672,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":1,"revision":"b9f6a527c7cb91c44aad808f9bf1ff3097d3596b","parents":["53f8829b05bdc300877ee1078da57f686132806d"],"ref":"refs/changes/01/1/1","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686214672,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"repo1","branch":"master","id":"I54ac7b3929a6487a697d4a5e8a8768469a3dd89a","number":1,"subject":"this is a test","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/repo1/+/1","commitMessage":"this is a test\n\nChange-Id: I54ac7b3929a6487a697d4a5e8a8768469a3dd89a\n","createdOn":1686214672,"status":"NEW","wip":true},"project":"repo1","refName":"refs/heads/master","changeKey":{"id":"I54ac7b3929a6487a697d4a5e8a8768469a3dd89a"},"type":"patchset-created","eventCreatedOn":1686214672,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"repo1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686214672,"instanceId":"replica-1"}
```

**Clone project project1 / Checkout change / Add a file and push to change**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"6210facff71d1cab8917b981dc9fb8b98fb8a167","refName":"refs/changes/01/1/3","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686215195,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"2087ce3d4971f3ab98603bf546a9a17b9c134b35","newRev":"07be7059d2596f418d19b5f785f61a448055ed05","refName":"refs/changes/01/1/meta","project":"repo1"},"type":"ref-updated","eventCreatedOn":1686215195,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":3,"revision":"6210facff71d1cab8917b981dc9fb8b98fb8a167","parents":["53f8829b05bdc300877ee1078da57f686132806d"],"ref":"refs/changes/01/1/3","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686215195,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"repo1","branch":"master","id":"I54ac7b3929a6487a697d4a5e8a8768469a3dd89a","number":1,"subject":"this is a test","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/repo1/+/1","commitMessage":"this is a test\n\nChange-Id: I54ac7b3929a6487a697d4a5e8a8768469a3dd89a\n","createdOn":1686214672,"status":"NEW","wip":true},"project":"repo1","refName":"refs/heads/master","changeKey":{"id":"I54ac7b3929a6487a697d4a5e8a8768469a3dd89a"},"type":"patchset-created","eventCreatedOn":1686215195,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"repo1","ref":"refs/changes/01/1/3","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686215195,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"FAST_FORWARD","project":"repo1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686215195,"instanceId":"replica-1"}
```

**Stop primary2**
no events

**Create project project3 / Create branch branch1 in project3**
no relevant

**Restart primary2**

- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215432,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215432,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"repo1","ref":"..all..","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215432,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686215433,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"repo1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686215433,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Projects","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686215433,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"repo1","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686215433,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686215433,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Users","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686215433,"instanceId":"replica-1"}
```

**ssh command to start pull-replication in primary2 with options: --all --wait**

- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215589,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215589,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"repo1","ref":"..all..","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686215589,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686215590,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"repo1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/repo1.git","type":"fetch-ref-replicated","eventCreatedOn":1686215590,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686215590,"instanceId":"replica-1"}
```

### Scenario 2 -  primary1/primary2 stable-3.5 with pull-replication and maxApiPayloadSize=1

**Create project project1**
- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"bd1c8245ba0c743b2add015f370045bfd82fef86","refName":"refs/meta/config","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216078,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"49233a11174a1363a9785ebed5e9970fe1c275ee","refName":"refs/heads/master","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216078,"instanceId":"primary"}
"type":"project-created" =>     {"projectName":"project1","headName":"refs/heads/master","type":"project-created","eventCreatedOn":1686216078,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216078,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/master","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216078,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/master","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216079,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216079,"instanceId":"replica-1"}
```

**Delete project project2**
no events

**Create branch branch1 in project1**
- Events in primary1:
```
"type":"ref-updated" =>     {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"49233a11174a1363a9785ebed5e9970fe1c275ee","refName":"refs/heads/branch1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216170,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/branch1","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216170,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/branch1","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216171,"instanceId":"replica-1"}
```

**Create a change in project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"cd22d76a6ea558cc11a5c96050ebc34cf47daec8","refName":"refs/changes/01/1/1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216233,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"3581322b75646ad80d18ee2be8481eb9397e2a8b","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216233,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":1,"revision":"cd22d76a6ea558cc11a5c96050ebc34cf47daec8","parents":["49233a11174a1363a9785ebed5e9970fe1c275ee"],"ref":"refs/changes/01/1/1","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686216233,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I9fc6365ff80096bce747e4e36395933b864a51cd","number":1,"subject":"asdasd","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"asdasd\n\nChange-Id: I9fc6365ff80096bce747e4e36395933b864a51cd\n","createdOn":1686216233,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I9fc6365ff80096bce747e4e36395933b864a51cd"},"type":"patchset-created","eventCreatedOn":1686216234,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/1","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216234,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216234,"instanceId":"replica-1"}
```

**Clone project project1 / Checkout change / Add a file and push to change**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"2ba527301c44e790fd2502469ada1f262beb5360","refName":"refs/changes/01/1/2","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216408,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"3581322b75646ad80d18ee2be8481eb9397e2a8b","newRev":"2dda3a71b053ba32b6c0980ead0b1f174e740466","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686216408,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":2,"revision":"2ba527301c44e790fd2502469ada1f262beb5360","parents":["49233a11174a1363a9785ebed5e9970fe1c275ee"],"ref":"refs/changes/01/1/2","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686216408,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I9fc6365ff80096bce747e4e36395933b864a51cd","number":1,"subject":"asdasd","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"asdasd\n\nChange-Id: I9fc6365ff80096bce747e4e36395933b864a51cd\n","createdOn":1686216233,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I9fc6365ff80096bce747e4e36395933b864a51cd"},"type":"patchset-created","eventCreatedOn":1686216408,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/2","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216408,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/meta","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216408,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/2","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216409,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"FAST_FORWARD","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216409,"instanceId":"replica-1"}
```

**Stop primary2**
no events

**Create project project3 / Create branch branch1 in project3**
no relevant

**Restart primary2**

- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216611,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216611,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216611,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216611,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project1","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Users","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project2","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686216612,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Projects","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686216612,"instanceId":"replica-1"}
```

**ssh command to start pull-replication in primary2 with options: --all --wait**
- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216770,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216770,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216770,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686216770,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686216772,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686216772,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686216772,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686216772,"instanceId":"replica-1"}
```

### Scenario 3 -  primary1/primary2 stable-3.5 with pull-replication, maxApiPayloadSize=40000 and events-kafka

**Create project project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"ca3a272c72a51fa0ca72c5c815c76a40a2f78b43","refName":"refs/meta/config","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217501,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"8f869753038762114788363c29184a73b61eee99","refName":"refs/heads/master","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217501,"instanceId":"primary"}
"type":"project-created" =>     {"projectName":"project1","headName":"refs/heads/master","type":"project-created","eventCreatedOn":1686217501,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217501,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/meta/config","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217501,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/meta/config","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217501,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/master","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217501,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/master","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217501,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/meta/config","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217502,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/meta/config","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217502,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217502,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/heads/master","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217502,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/heads/master","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217502,"instanceId":"replica-1"}
```

**Delete project project2**
no events

**Create branch branch1 in project1**

- Events in primary1:
```
"type":"ref-updated" =>     {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"8f869753038762114788363c29184a73b61eee99","refName":"refs/heads/branch1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217579,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/branch1","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217579,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/branch1","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217579,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/heads/branch1","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217580,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/heads/branch1","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217580,"instanceId":"replica-1"}
```

**Create a change in project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"261ef4658dfaa07585bf0fe636052870b379dd59","refName":"refs/changes/01/1/1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217630,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"f2763500b159e20dbc555a1a9a8969439a0c3a97","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217630,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":1,"revision":"261ef4658dfaa07585bf0fe636052870b379dd59","parents":["8f869753038762114788363c29184a73b61eee99"],"ref":"refs/changes/01/1/1","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686217630,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I88bb6c8d40da9361b16cb456b9f020cd81dc4b87","number":1,"subject":"asdad","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"asdad\n\nChange-Id: I88bb6c8d40da9361b16cb456b9f020cd81dc4b87\n","createdOn":1686217630,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I88bb6c8d40da9361b16cb456b9f020cd81dc4b87"},"type":"patchset-created","eventCreatedOn":1686217631,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217630,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/meta","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217631,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/1","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217631,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/meta","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217631,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/1","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217631,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/meta","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217631,"instanceId":"replica-1"}
```

**Clone project project1 / Checkout change / Add a file and push to change**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"a2e74f67809e04dcf76cb026c76154a579b0ea4f","refName":"refs/changes/01/1/2","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217812,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"f2763500b159e20dbc555a1a9a8969439a0c3a97","newRev":"bf559ee943cbade8b3bff1cc99a4d2d27865027a","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686217812,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":2,"revision":"a2e74f67809e04dcf76cb026c76154a579b0ea4f","parents":["8f869753038762114788363c29184a73b61eee99"],"ref":"refs/changes/01/1/2","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686217812,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I88bb6c8d40da9361b16cb456b9f020cd81dc4b87","number":1,"subject":"asdad","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"asdad\n\nChange-Id: I88bb6c8d40da9361b16cb456b9f020cd81dc4b87\n","createdOn":1686217630,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I88bb6c8d40da9361b16cb456b9f020cd81dc4b87"},"type":"patchset-created","eventCreatedOn":1686217812,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/2","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217812,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/2","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217812,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"FAST_FORWARD","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217812,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/meta","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217812,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/2","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217813,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/meta","status":"not-attempted","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217813,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/2","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217813,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"project":"project1","ref":"refs/changes/01/1/meta","status":"failed","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217813,"instanceId":"replica-1"}
```

**Stop primary2**
no events

**Create project project3 / Create branch branch1 in project3**
no relevant

**Restart primary2**

- Events in primary1:
No events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"..all..","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217960,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"refs/heads/branch1","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217960,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project3","ref":"refs/heads/branch1","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686217961,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project3","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686217961,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217986,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217986,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217986,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217986,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"..all..","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686217986,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Projects","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Users","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project1","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project2","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project3","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686217987,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>  {   "project":"project3","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686217987,"instanceId":"replica-1"}
```

**ssh command to start pull-replication in primary2 with options: --all --wait**

- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686218018,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686218018,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686218018,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686218018,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"..all..","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686218018,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686218019,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686218019,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686218019,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686218019,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project3","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686218019,"instanceId":"replica-1"}
```

### Scenario 4 -  primary1/primary2 stable-3.5 with pull-replication, maxApiPayloadSize=1 and events-kafka

**Create project project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"e15a022830ce865198b87869d0711e0001bd013f","refName":"refs/meta/config","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225227,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"cda029b4a012032b217f78576b947e5181cee7d0","refName":"refs/heads/master","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225227,"instanceId":"primary"}
"type":"project-created" =>     {"projectName":"project1","headName":"refs/heads/master","type":"project-created","eventCreatedOn":1686225227,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225227,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/meta/config","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225227,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/master","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225227,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/master","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225227,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/master","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225228,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/master","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225228,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/meta/config","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225228,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225228,"instanceId":"replica-1"}
```

**Delete project project2**
no events

**Create branch branch1 in project1**

- Events in primary1:
```
"type":"ref-updated" =>     {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"cda029b4a012032b217f78576b947e5181cee7d0","refName":"refs/heads/branch1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225301,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/heads/branch1","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225301,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/heads/branch1","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225302,"instanceId":"replica-1"}
```

**Create a change in project1**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"c56f34740376304013082d530833d2a5965d3460","refName":"refs/changes/01/1/1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225341,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"41b0d051d4a6a9ffa10a94e8977b53fa1d38a332","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225341,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":1,"revision":"c56f34740376304013082d530833d2a5965d3460","parents":["cda029b4a012032b217f78576b947e5181cee7d0"],"ref":"refs/changes/01/1/1","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686225341,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I3f97952441de4dc6818839ec7eacc988afe8de54","number":1,"subject":"adad","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"adad\n\nChange-Id: I3f97952441de4dc6818839ec7eacc988afe8de54\n","createdOn":1686225341,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I3f97952441de4dc6818839ec7eacc988afe8de54"},"type":"patchset-created","eventCreatedOn":1686225341,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/1","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225342,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>    {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225342,"instanceId":"replica-1"}
```
**Clone project project1 / Checkout change / Add a file and push to change**

- Events in primary1:
```
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"27dbe90396388e227b23f183430d3af445e8af3a","refName":"refs/changes/01/1/2","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225534,"instanceId":"primary"}
"type":"ref-updated" =>         {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"41b0d051d4a6a9ffa10a94e8977b53fa1d38a332","newRev":"34417feab6f965e0223f1309ccd655558766ac6e","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686225534,"instanceId":"primary"}
"type":"patchset-created" =>    {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":2,"revision":"27dbe90396388e227b23f183430d3af445e8af3a","parents":["cda029b4a012032b217f78576b947e5181cee7d0"],"ref":"refs/changes/01/1/2","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686225534,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I3f97952441de4dc6818839ec7eacc988afe8de54","number":1,"subject":"adad","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"adad\n\nChange-Id: I3f97952441de4dc6818839ec7eacc988afe8de54\n","createdOn":1686225341,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I3f97952441de4dc6818839ec7eacc988afe8de54"},"type":"patchset-created","eventCreatedOn":1686225534,"instanceId":"primary"}
```

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/2","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225534,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"refs/changes/01/1/meta","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225534,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NEW","project":"project1","ref":"refs/changes/01/1/2","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225535,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"FAST_FORWARD","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225535,"instanceId":"replica-1"}
```

**Stop primary2**
no events

**Create project project3 / Create branch branch1 in project3**
no relevant

**Restart primary2**

- Events in primary1:
no events:

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225762,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225762,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225762,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225762,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"..all..","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225762,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Projects","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"All-Users","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project1","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project2","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project3","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686225763,"instanceId":"replica-1"}
"type":"fetch-ref-replication-done" =>      {"project":"project3","ref":"..all..","nodesCount":1,"type":"fetch-ref-replication-done","eventCreatedOn":1686225763,"instanceId":"replica-1"}
```

**ssh command to start pull-replication in primary2 with options: --all --wait**

- Events in primary1:
no events

- Events in primary2:
```
"type":"fetch-ref-replication-scheduled" => {"project":"All-Projects","ref":"..all..","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225828,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"All-Users","ref":"..all..","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225828,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project1","ref":"..all..","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225828,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project2","ref":"..all..","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225828,"instanceId":"replica-1"}
"type":"fetch-ref-replication-scheduled" => {"project":"project3","ref":"..all..","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replication-scheduled","eventCreatedOn":1686225828,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project2","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project2.git","type":"fetch-ref-replicated","eventCreatedOn":1686225829,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Projects","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Projects.git","type":"fetch-ref-replicated","eventCreatedOn":1686225829,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"All-Users","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/All-Users.git","type":"fetch-ref-replicated","eventCreatedOn":1686225829,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project1","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project1.git","type":"fetch-ref-replicated","eventCreatedOn":1686225829,"instanceId":"replica-1"}
"type":"fetch-ref-replicated" =>            {"refUpdateResult":"NO_CHANGE","project":"project3","ref":"..all..","status":"succeeded","targetUri":"http://gerrit1:8080/project3.git","type":"fetch-ref-replicated","eventCreatedOn":1686225829,"instanceId":"replica-1"}
```

### Scenario 5 -  primary1/primary2 stable-3.5 with push replication

**Create project project1**

- Events in primary1:
```
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/meta/config","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239143}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"6987643e9cb5a57e1cbe1151c2ae7cd7ac080da2","refName":"refs/meta/config","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239143}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/heads/master","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239143}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"75ac17c31fbd3c31f3b951b06c978b4718a2020e","refName":"refs/heads/master","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239143}
"type":"project-created" =>             {"projectName":"project1","headName":"refs/heads/master","type":"project-created","eventCreatedOn":1686239143}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"NON_EXISTING","project":"project1","ref":"refs/meta/config","status":"failed","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239143}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"NON_EXISTING","project":"project1","ref":"refs/heads/master","status":"failed","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239143}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/meta/config","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239143}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/heads/master","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239143}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/meta/config","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239143}
"type":"ref-replication-done" =>        {"project":"project1","ref":"refs/meta/config","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239143}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/heads/master","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239143}
"type":"ref-replication-done" =>        {"project":"project1","ref":"refs/heads/master","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239143}
```

- Events in primary2:
no events

**Create branch branch1 in project1**

- Events in primary1:
```
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/heads/branch1","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239216}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"75ac17c31fbd3c31f3b951b06c978b4718a2020e","refName":"refs/heads/branch1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239216}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/heads/branch1","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239216}
"type":"ref-replication-done" =>        {"project":"project1","ref":"refs/heads/branch1","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239216}
```

- Events in primary2:
no events

**Create a change in project1**

- Events in primary1:
```
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/changes/01/1/meta","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239243}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/changes/01/1/1","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239243}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"196c578ac80bb636fa95c3a7c5d1b9c2f6bedd4b","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239243}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"6083117f4051283d716075171774a69497a698f6","refName":"refs/changes/01/1/1","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239243}
"type":"patchset-created" =>            {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":1,"revision":"6083117f4051283d716075171774a69497a698f6","parents":["75ac17c31fbd3c31f3b951b06c978b4718a2020e"],"ref":"refs/changes/01/1/1","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686239243,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I611d1a9517263afed28cdea0bafcb3f12f06b5fe","number":1,"subject":"ada","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"ada\n\nChange-Id: I611d1a9517263afed28cdea0bafcb3f12f06b5fe\n","createdOn":1686239243,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I611d1a9517263afed28cdea0bafcb3f12f06b5fe"},"type":"patchset-created","eventCreatedOn":1686239243}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239243}
"type":"ref-replication-done" =>        {"project":"project1","ref":"refs/changes/01/1/meta","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239243}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/changes/01/1/1","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239243}
"type":"ref-replication-done" =>        {"project":"project1","ref":"refs/changes/01/1/1","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239243}
```

- Events in primary2:
no evnets

**Clone project project1 / Checkout change / Add a file and push to change**

- Events in primary1:
```
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/changes/01/1/meta","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239428}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"refs/changes/01/1/2","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239428}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"0000000000000000000000000000000000000000","newRev":"02cdfa52fc26fa2a6dd98d911d514168d835800f","refName":"refs/changes/01/1/2","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239428}
"type":"ref-updated" =>                 {"submitter":{"name":"Administrator","email":"admin@example.com","username":"admin"},"refUpdate":{"oldRev":"196c578ac80bb636fa95c3a7c5d1b9c2f6bedd4b","newRev":"e2ed846c9da278684d77773de730e6f944a79b7c","refName":"refs/changes/01/1/meta","project":"project1"},"type":"ref-updated","eventCreatedOn":1686239428}
"type":"patchset-created" =>            {"uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"patchSet":{"number":2,"revision":"02cdfa52fc26fa2a6dd98d911d514168d835800f","parents":["75ac17c31fbd3c31f3b951b06c978b4718a2020e"],"ref":"refs/changes/01/1/2","uploader":{"name":"Administrator","email":"admin@example.com","username":"admin"},"createdOn":1686239428,"author":{"name":"Administrator","email":"admin@example.com","username":"admin"},"kind":"REWORK","sizeInsertions":9,"sizeDeletions":0},"change":{"project":"project1","branch":"master","id":"I611d1a9517263afed28cdea0bafcb3f12f06b5fe","number":1,"subject":"ada","owner":{"name":"Administrator","email":"admin@example.com","username":"admin"},"url":"http://localhost:8080/c/project1/+/1","commitMessage":"ada\n\nChange-Id: I611d1a9517263afed28cdea0bafcb3f12f06b5fe\n","createdOn":1686239243,"status":"NEW","wip":true},"project":"project1","refName":"refs/heads/master","changeKey":{"id":"I611d1a9517263afed28cdea0bafcb3f12f06b5fe"},"type":"patchset-created","eventCreatedOn":1686239428}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/changes/01/1/meta","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239428}
"type":"ref-replication-done" =>       {"project":"project1","ref":"refs/changes/01/1/meta","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239428}
"type":"ref-replicated" =>              {"targetNode":"/var/gerrit/git-replica/project1.git","refStatus":"OK","project":"project1","ref":"refs/changes/01/1/2","status":"succeeded","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replicated","eventCreatedOn":1686239428}
"type":"ref-replication-done" =>       {"project":"project1","ref":"refs/changes/01/1/2","nodesCount":1,"type":"ref-replication-done","eventCreatedOn":1686239428}
```

- Events in primary2:
no events

**Stop primary2**
no events

**Create project project3 / Create branch branch1 in project3**
no relevant

**Restart primary2**
no events

**ssh command to start replication in primary1 with options: --all --wait**
- Events in primary1:
```
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/All-Projects.git","project":"All-Projects","ref":"..all..","targetUri":"file:///var/gerrit/git-replica/All-Projects.git","type":"ref-replication-scheduled","eventCreatedOn":1686239913}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/All-Users.git","project":"All-Users","ref":"..all..","targetUri":"file:///var/gerrit/git-replica/All-Users.git","type":"ref-replication-scheduled","eventCreatedOn":1686239913}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/ecample.git","project":"ecample","ref":"..all..","targetUri":"file:///var/gerrit/git-replica/ecample.git","type":"ref-replication-scheduled","eventCreatedOn":1686239913}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/p3.git","project":"p3","ref":"..all..","targetUri":"file:///var/gerrit/git-replica/p3.git","type":"ref-replication-scheduled","eventCreatedOn":1686239913}
"type":"ref-replication-scheduled" =>   {"targetNode":"/var/gerrit/git-replica/project1.git","project":"project1","ref":"..all..","targetUri":"file:///var/gerrit/git-replica/project1.git","type":"ref-replication-scheduled","eventCreatedOn":1686239913}
```

- Events in primary2:
no events
