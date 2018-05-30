Service discovery pattern
=========================

Let's assume we have service SVC that needs to be run and registered in Service Discovery service SD.
The health checks will be performed by Sidecar Service SCSVC that will periodically check the status of
SVC and keep it registered in SD. SCSVC will be also responsible for unregistering of SVC from SD in
case the SVC is manually stopped.

I try to implement it using systemd unit files.

SVC unit file
-------------
    $ cat svc.service
    [Unit]
    Requires=scsvc.service
    Before=scsvc.service

    [Service]
    Type=simple
    ExecStart=/usr/bin/runsvc start
    ExecStop=/usr/bin/runsvc stop

SCSVC unit file
---------------
    $ cat scsvc.service
    [Unit]
    Requires=svc.service
    After=svc.service
    BindsTo=svc.service

    [Service]
    ExecStartPre=/usr/bin/sidecar wait_on svc
    ExecStartPre=/usr/bin/sidecar register svc
    ExecStart=/usr/bin/sidecar check svc
    ExecStop=/usr/bin/sidecar unregister svc
    ExecStop=/usr/bin/sidecar stop svc

