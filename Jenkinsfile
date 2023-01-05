
@Library('jenkins-shared-libraries/vmware') _
def  physicalHosts    = []
def  hostDatastoreMap = [:]
def  hostNetworkMap   = [:]

pipeline{
    
    agent{
        
        docker{
            image 'vmware-powercli-tools'
            args  '-u 0'
            label 'linux'
        }
    }
	parameters{
	 string(name:'Name', defaultValue:'', description:'The name of the virtual server  (if  only one server is to be created ) or the prefix of the names of the virtual server to be created ( numbers would be appended sequentially to the prefix starting from 1)')
	 choice(name:'OS', choices: ['Windows','Linux'], description: 'Specify the operating system type of each server')
	 choice(name:'Count',choices: [1,2,3,4,5,6,7,8,9,10], description:'The number of servers to be created')
	 choice(name:'DiskCount',choices: [1,2,3,4],description: 'The  number of hard drives to be added to each server')	
     choice(name:'DiskSize',choices: [100,150,200,250,300,350,400,450,500,550,600,650,700,750,800,850,900,950,1000,1050,1100,1150,1200,1250,1300,1350,1400,1450,1500],description: 'The size of each hard drive in GB to be added to the server')	
	 choice(name:'CPU',choices: [1,2,4,6,8,10,12,14,16,18,20,22,24,26], description:'The number of virtual cpus to be added to each server created')
     choice(name:'RAM',choices: [2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58,60,62,64], description:'The size of the RAM (physical memory) in GB of each server to be created')	
	 choice(name:'NetworkInterfaces',choices: [1,2,3,4], description:'The number of NICs (Network Interface cards to be created) for each server')	
	 choice(name:'Environment',choices: ['Test','Production'],description:'The environment where the servers are to be created')
	 choice(name:'UseDefaults',choices: ['YES','NO'], description:"Select 'NO' to make changes to some additional/advanced VM configurations.")
	}
    environment{    
       vCenterServer7='172.22.38.56' 
       vCenterCred7 = credentials('VCenter7.0')
       vCenterServer6='172.22.38.50' 
       vCenterCred6 = credentials('vSphere_6.7')
    }
    stages{
        
    stage('Connect-to-VCenter'){
        
        steps{
            script{
             def  vmInformation    = getVMInfo(params.Environment)
           
             vmInformation['hosts'].each{

                physicalHosts.add(it.Name)
            
             }
             def previousHost = null
             def datastoreArray = []
            vmInformation['hostDatastoreMap'].each{

                 if(!previousHost||(previousHost==it.HostName)){
                    datastoreArray.push( it.DataStoreName)
                 }else{
                    hostDatastoreMap[previousHost] = datastoreArray
                    datastoreArray =[]
                    datastoreArray.push( it.DataStoreName)
                 }   
                 previousHost=it.HostName            
             }

             previousHost = null
             def networkArray = []
             vmInformation['hostNetworkMap'].each{

                 if(!previousHost||(previousHost==it.HostName)){
                    networkArray.push( it.NetworkName)
                 }else{
                    hostNetworkMap[previousHost] = networkArray
                    networkArray =[]
                     networkArray.push( it.NetworkName)
                 }   
                 previousHost=it.HostName            
             }


             def vmSeparatorHeaderStyle = """
                background-color: #3c54cf;
                text-align: center;
                padding: 4px;
                color: #fffff;
                font-size: 18px;
                font-weight: normal;
                text-transform: uppercase;
                font-family: consolas,'Orienta', sans-serif;
                letter-spacing: 1px;
                font-style: bold;
             """

             def subHeaderStyle  ="""
                text-align: lef;
                padding: 2px;
                color: #fffff;
                font-size: 16px;
                font-weight: normal;
                text-transform: uppercase;"""
             String vmName         =  params.Name.toString()
             int vmCount           =  Integer.parseInt(params.Count)
             int diskCount         =  Integer.parseInt(params.DiskCount)
             int ramSizePerVM      =  Integer.parseInt(params.RAM)
             int cpuSizePerVM      =  Integer.parseInt(params.CPU)
             int diskSizePerVM     =  Integer.parseInt(params.DiskSize)
             int nicCount          =  Integer.parseInt(params.NetworkInterfaces)

             int totalDiskSize     =  diskSizePerVM * vmCount
             int totalCpu          =  cpuSizePerVM  * vmCount
             int totalMemory       =  ramSizePerVM  * vmCount
             
             //echo "totalDiskSize: ${totalDiskSize}, totalCpu:${totalCpu}, totalMemory:${totalMemory} "
             
            /*  vmInformation.each{
                 echo it.toString()
             }
             */

		    def updatedVMSpecifications  = [];
            def vmConfigInputParameters   = []
            
            if(params.UseDefaults.toLowerCase()=="yes"){ 
                for( def k=0; k<vmCount; k++){
                        def natIndex = k+1
                        def vmSpecsMap = [:];
                        vmSpecsMap["name"]                  = natIndex.toString().length==1?"${vmName}00${natIndex}":"${vmName}0${natIndex}"
                        vmSpecsMap["nmCpu"]                 = params.CPU
                        vmSpecsMap["memoryGb"]              = params.RAM
                        vmSpecsMap["networkName"]           = 'VM Network'
                        vmSpecsMap["hasCD"]                 = true
                        vmSpecsMap["hasFloppy"]             = false
                        vmSpecsMap["cluster"]               = ""
                        vmSpecsMap["osDistribution"]        = params.OS
                        vmSpecsMap["ipMode"]                = "dhcp"
                        vmSpecsMap["ip"]                    = null
                        vmSpecsMap["mask"]                  = null
                        vmSpecsMap["gateway"]               = null
                        vmSpecsMap["dns"]                   = null
                        if (params.Environment              == "production"){ 
                            vmSpecsMap["template"]          = params.OS.toString().toLowerCase() == "windows"? "CMTeam_Win10x64": "CMTeam_U2004x64_Template"
                            vmSpecsMap["storageFormat"]     =  "thick"
                        }else if (params.Environment        == "test"){ 
                            vmSpecsMap["template"]          =   params.OS.toString().toLowerCase() == "windows"? "CMTeam_Win10x64": "CMTeam_U2004x64_Template"
                            vmSpecsMap["storageFormat"]     =   "thin"
                        }
                        updatedVMSpecifications.Add(vmSpecsMap);
                }
                    
            }else if(params.UseDefaults.toLowerCase()=="no"){ 
                
                def vmHostMap = getVMHostMap(vmName,vmCount,physicalHosts)

               vmHostMap = readJSON (text :vmHostMap)
                echo vmHostMap
                for( def k=0; k<vmCount; k++){
                    def indexNatural = k+1
                    def vmIndex= indexNatural
                    def vmSpecsMap = [:]
                    def vmID = "Server${indexNatural}"
                    def serverName = indexNatural <9 ?"${vmName}00${indexNatural}":"${vmName}0${indexNatural}"
                    //def vmHostName = vmHostMap[vmID]

                   // echo "${vmHostName} has been chosed as the host of  ${serverName}"
                   /*
                    vmConfigInputParameters.addAll(
                        [
                        separator(name: serverName, sectionHeader: "${serverName} Server Specifications",separatorStyle: "border-width: 0", sectionHeaderStyle: vmSeparatorHeaderStyle)
                        ,string(name:"${serverName}_Name", defaultValue: serverName, description:'Choose a custom name for the new server')
                        ,choice(name:"${serverName}_OS", choices: ['Windows','Linux'], description: "Specify the operating system for ${serverName}")
                        ,choice(name:"${serverName}_PhysicalHost", choices: physicalHosts, description: "Specify the physical host for ${serverName}")
                        ]
                    )

                    for (def i = 0; i<diskCount; i++){
                        def index =  i+1
                        vmConfigInputParameters.addAll(
                        [   
                            separator(name: "${serverName}_Disk${i}_separator", sectionHeader: "${serverName}_Disk${index} Details",sectionHeaderStyle: subHeaderStyle),
                            ,choice(name:"${serverName}_Disk${index}_Size",choices: [100,150,200,250,300,350,400,450,500,550,600,650,700,750,800,850,900,950,1000,1050,1100,1150,1200,1250,1300,1350,1400,1450,1500],description: 'The size of each hard drive in GB to be added to the server')	
                            ,[$class: 'CascadeChoiceParameter', choiceType: 'PT_SINGLE_SELECT',description:  "The Datastore for the hard disk ${index} of ${serverName} ",filterLength: 1, filterable: true,
                                name: "${serverName}_Disk${index}_Location"
                                ,referencedParameters: "${serverName}_PhysicalHost"
                                ,script: [$class: 'GroovyScript',  
                                            fallbackScript: [
                                            classpath: [],  sandbox: true, 
                                            script: """
                                            return ["Fallback"]
                                            """.stripIndent()
                                            ] ,
                                            script:[
                                                classpath: [],   sandbox: true, 
                                                script: """
                                                   return ["${params.each{it.key}}"]   
                                                """.stripIndent()
                                            ]
                                        ]
                            ]
                        
                        ]
                        )
                    }

                      for (def i = 0; i<nicCount; i++){
                             def index =  i+1
                            vmConfigInputParameters.addAll(
                            [   
                                separator(name: "${serverName}_NIC${i}_separator", sectionHeader: "${serverName}_NIC${index} Details",sectionHeaderStyle: subHeaderStyle),
                                 ,[$class: 'CascadeChoiceParameter', choiceType: 'PT_SINGLE_SELECT',description:  "The Network PortGroup for NIC ${index} of ${serverName} ",filterLength: 1, filterable: true,
                                    name: "${serverName}_NIC${index}",   referencedParameters: "${serverName}_PhysicalHost",
                                    script: [$class: 'GroovyScript',  
                                                fallbackScript: [
                                                classpath: [],  sandbox: true, 
                                                script: """
                                                return ["Fallback"]
                                                """.stripIndent()
                                                ] ,
                                                script:[
                                                    classpath: [],   sandbox: true, 
                                                    script: """
                                                           def host       = "${serverName}_PhysicalHost"
                                                                              
                                                            return [parameter]
                                                    """.stripIndent()
                                                ]
                                            ]
                                ]
                            
                            ]
                            )

                      }

  */
                 } 
             //def vmSpecsModificationInput = input(id: 'vmSpecsModificationInput', message:'Click this link to provide additional information', parameters: vmConfigInputParameters, ok:'Provision')
           
            }

                //to be set during deployment
                //    vmSpecsMap["vmHost"]                = ""
                //    vmSpecsMap["diskSummary"]           = ""
             
            }

             
              /*  
			if(params.UseDefaults.toLowerCase()=="yes"){ 
             
                    if (params.Environment == "production"){ 
                        updatedVMSpecifications["template"] = params.OS.toString().toLowerCase() == "windows"? "CMTeam_Win10x64": "CMTeam_U2004x64_Template"
                        updatedVMSpecifications["storageFormat"]="thick"
                    }else if (params.Environment== "test"){ 
                        updatedVMSpecifications["template"] = params.OS.toString().toLowerCase() == "windows"? "CMTeam_Win10x64": "CMTeam_U2004x64_Template"
                        updatedVMSpecifications["storageFormat"]="thin"
                    }
                    
            }else   if(params.UseDefaults.toLowerCase()=="no"){ 


            }


           
                DiskCount=1
                OS=Windows
                NetworkInterfaces=1
                UseDefaults=YES
                DiskSize=100
                CPU=1
                Environment=Test
                Count=1
                Name=TESTVM
                RAM=2
            */
        
        }
    }
  
    }
}