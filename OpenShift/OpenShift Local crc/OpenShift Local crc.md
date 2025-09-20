
`~ » crc start`
`INFO Using bundle path /Users/jwon/.crc/cache/crc_vfkit_4.19.8_arm64.crcbundle` 
`INFO Checking if running macOS version >= 13.x`    
`INFO Checking if running as non-root`              
`INFO Checking if crc-admin-helper executable is cached` 
`INFO Checking if running on a supported CPU architecture` 
`INFO Checking if crc executable symlink exists`    
`INFO Checking minimum RAM requirements`            
`INFO Check if Podman binary exists in: /Users/jwon/.crc/bin/oc` 
`INFO Checking if running emulated on Apple silicon` 
`INFO Checking if vfkit is installed`               
`INFO Checking if old launchd config for tray and/or daemon exists` 
`INFO Checking if crc daemon plist file is present and loaded` 
`INFO Checking SSH port availability`               
`INFO Loading bundle: crc_vfkit_4.19.8_arm64...`    
`INFO Starting CRC VM for openshift 4.19.8...`      
`INFO CRC instance is running with IP 127.0.0.1`    
`INFO CRC VM is running`                            
`INFO Updating authorized keys...`                  
`INFO Configuring shared directories`               
`INFO Check internal and public DNS query...`       
`INFO Check DNS query from host...`                 
`INFO Verifying validity of the kubelet certificates...` 
`INFO Starting kubelet service`                     
`INFO Waiting for kube-apiserver availability... [takes around 2min]` 
`INFO Waiting until the user's pull secret is written to the instance disk...` 
`INFO Overriding password for developer user`       
`INFO Starting openshift instance... [waiting for the cluster to stabilize]` 
`INFO Operator network is progressing`              
`INFO 3 operators are progressing: console, ingress, network` 
`INFO Operator ingress is progressing`              
`INFO All operators are available. Ensuring stability...` 
`INFO Operators are stable (2/3)...`                
`INFO Operators are stable (3/3)...`                
`INFO Adding crc-admin and crc-developer contexts to kubeconfig...` 
`Started the OpenShift cluster.`

`The server is accessible via web console at:`
  `https://console-openshift-console.apps-crc.testing`

`Log in as administrator:`
  `Username: kubeadmin`
  `Password: vRevF-xKTzK-wUgWk-MXN6n`

`Log in as user:`
  `Username: developer`
  `Password: developer`

`Use the 'oc' command line interface:`
  `$ eval $(crc oc-env)`
  `$ oc login -u developer https://api.crc.testing:6443`