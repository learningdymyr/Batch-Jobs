mkdir /usr/share/jobdemo
nano /usr/share/jobdemo/workqueue
1
2
3
4
5
6
7
8
9
10
11
12

cp /usr/share/jobdemo/workqueue /usr/share/jobdemo/workqueue-backup
nano /usr/share/jobdemo/jobscript.sh
#!/bin/bash
for counter in 1 2 3 4 5 6 7 8 9 10 11 12
do
if [ -s /usr/share/jobdemo/workqueue ]
then 
   echo " did some work "
   sed -i '1d' /usr/share/jobdemo/workqueue
   sleep 1
else
   echo " no more work left "
   exit 0
fi
done
exit 0


====
nano myPersistent-Volume.yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: my-persistent-volume
  labels:
    type: local
spec:
  storageClassName: pv-demo 
  capacity:
    storage: 10Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/usr/share/jobdemo"
===
nano myPersistent-VolumeClaim.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: my-persistent-volumeclaim
spec:
  storageClassName: pv-demo 
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
===
kubectl create -f myPersistent-Volume.yaml
kubectl create -f myPersistent-VolumeClaim.yaml

nano myWorkqueue-Job.yaml
AS per your reference
https://pastebin.com/wMc2mQQ3
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  parallelism: 1
  template:
    spec:
      containers:
        - name: myjob
          image: alpine
          imagePullPolicy: IfNotPresent
          command: ['sh', '-c', 'source /usr/share/jobdemo/jobscript.sh']
          volumeMounts:
          - mountPath: "/usr/share/jobdemo"
            name: my-persistent-volumeclaim-name
      restartPolicy: Never
      terminationGracePeriodSeconds: 0
      volumes:
      - name: my-persistent-volumeclaim-name
        persistentVolumeClaim:
          claimName: my-persistent-volumeclaim

====
kubectl create -f myWorkqueue-Job.yaml
kubectl get pod


error:
=====
myjob-5d4rw   0/1     ContainerCannotRun   0          3s

kubectl describe pod <>

error:

Warning  Failed     72s   kubelet, gke-standard-cluster-1-default-pool-8beac272-s392  Error: failed to start container "myjob": Error response from daemon: error while creating mount source path '/usr/share/jobdemo': mkdir /usr/share/jobdemo: read-only file system


