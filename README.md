# openshift-tips

## Checking Health Checks  of all projects in openshift at once

I've got to verify health checks (liveness and readiness times) from all projects. It'll take too much time on the web interface so CLI comes to save the day.

After log in, eval the following code:

```
for i in `oc get projects | cut -f 1 -d ' '`; do
    echo $i ;
    oc get dc -o yaml -n $i  2> /dev/null | \
	egrep "(livenessProbe|readinessProbe)" -A 10 |  \
	egrep "(livenessProbe|readinessProbe|initialDelaySeconds|name)" | \
	sed -n -e '/Delay/!h' -e '/Delay/{x;G;s/\n//;p}' -e '/name/p' | \
	sed -n '/liveness/{h}; /name/{G;p}; /readiness/{p}'
done 
```
which will result in:

```
project1
          name: app1
          livenessProbe:            initialDelaySeconds: 420
          readinessProbe:            initialDelaySeconds: 3
project2
          name: app2
          livenessProbe:            initialDelaySeconds: 420
          readinessProbe:            initialDelaySeconds: 3
```




**if you have jq** (https://stedolan.github.io/jq/) you can do:

```
for i in `oc get projects | cut -f 1 -d ' '`; do
    echo $i ;
    oc get dc -o json -n $i | jq ' .items[].spec.template.spec.containers[] | {"app": .name, "liveness": .livenessProbe.initialDelaySeconds, "readiness": .readinessProbe.initialDelaySeconds}  '
done
```

resulting in a formatted output:
```
NAME
(...)
project
{
  "app": "appname",
  "liveness": 140,
  "readiness": 140
}
project2
{
  "app": "appname2",
  "liveness": 126,
  "readiness": 120
}
```
