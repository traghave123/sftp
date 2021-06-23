# sftp
# SFTP Centos Docker Image Creation  

1. Download the files in docker_files folder
2. Navigate to downloaded folder and run:  
    docker build -t [your image name]:[versionTag] .
3. Push this image to your docker registry:  
    docker push [your image name]:[versionTag]  
*//Optional Step:4 to test your docker container//*  
4. Docker run container:
    docker run --cap-add AUDIT_WRITE -v [hostPath]:[mountPath] -p [exposedPort]:[containerPort] -d [dockerimage]:[versionTag] [SFTPuser]:[password]:[uid]:[guid]:[directory]  
    optional params: uid,guid

# SFTP service containerized
Steps to be followed:  
1. Ensure to have docker image ready as mentioned in SFTP Centos Image Creation
2. Create namespace:  
    kubectl create namespace sftp  

*// Step 3: optional. Ignore if custom storage class is present and update this custom storage class name in the persistent_volume and pvolume_claim files //*  

 3. Create storage class:  
      oc apply -f storage_class.yaml  
    
*// Update the files according to your custom values if necessary before applying for the following steps //*  

4. Create persistent volume:  
    oc apply -f persistent_volume.yaml 
5. Create volume claim:  
    oc apply -f pvolume_claim.yaml
6. For privileged access run the command:  
    oc adm policy add-scc-to-user privileged -z default -n sftp  


7. Create deployment and service:  
      oc apply -f deployment.yaml

8. Configure loadbalancer :

    Edit /etc/haproxy/haproxy.cfg to add the entries:

    frontend fe_ssh
        bind *:2222 #exposed port
        mode tcp
        default_backend ssh-all
    backend ssh-all
        balance source
        mode tcp
        server master0 10.17.7.206:31336 check #internal IP of nodes:port of loadbalancer 
        server master1 10.17.16.156:31336 check
        server master2 10.17.16.173:31336 check
        server worker0 10.17.22.235:31336 check
        server worker1 10.17.23.98:31336 check
        server worker2 10.17.28.68:31336 check


9. Create route:  
    oc expose service [service name]  
    Eg: oc expose service sftp
10. Access sftp service through:  
    sftp -P [exposed port] [user]@[sftp-route]   
    Eg: sftp -P 2222 admin@sftp-rgv-lb-sftp.apps.raghavocp46.cp.fyre.ibm.com
