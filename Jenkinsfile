
@Library('esxi-provisioning-libraries') _
def  physicalHosts    = []
def  vmTemplates      = []
def  hostDatastoreMap = [:]
def  hostNetworkMap   = [:]


 properties([
    parameters([
     string(name:'ServerName', defaultValue:'', description:'The name of the virtual server  (if  only one server is to be created ) or the prefix of the names of the virtual server to be created ( numbers would be appended sequentially to the prefix starting from 1)')
	 ,choice(name:'OperatingSystem', choices: ['Windows','Linux'], description: 'Specify the operating system type of each server')
	 ,choice(name:'ServerCount',choices: [1,2,3,4,5,6,7,8,9,10], description:'The number of servers to be created')
	 ,choice(name:'DiskCount',choices: [1,2,3,4],description: 'The  number of hard drives to be added to each server')	
     ,choice(name:'DiskSize',choices: [100,150,200,250,300,350,400,450,500,550,600,650,700,750,800,850,900,950,1000,1050,1100,1150,1200,1250,1300,1350,1400,1450,1500],description: 'The size of each hard drive in GB to be added to the server')	
	 ,choice(name:'CPU',choices: [1,2,4,6,8,10,12,14,16,18,20,22,24,26], description:'The number of virtual cpus to be added to each server created')
     ,choice(name:'RAM',choices: [2,4,6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58,60,62,64], description:'The size of the RAM (physical memory) in GB of each server to be created')	
	 ,choice(name:'NetworkInterfaces',choices: [1,2,3,4], description:'The number of NICs (Network Interface cards to be created) for each server')	
	 ,choice(name:'Environment',choices: ['Test' ,'Production','Others'],description:'The VCenter server that manages the hosts where the server(s) will be created')
     ,[$class: 'DynamicReferenceParameter', 
        choiceType: 'ET_FORMATTED_HTML',
		omitValueField: true,
        description:'The VCenter server that manages the hosts where the server(s) will be created',
		randomName: 'string-parameter-01',
        name: 'VCenterServer',
        referencedParameters: 'Environment',
        script: [$class: 'GroovyScript',
            fallbackScript: [
                classpath: [], 
                sandbox: true, 
                script: """
						  inputBox="<input name='value' value='' type='text'>"
                """.stripIndent()
            ],
            script: [
                classpath: [], 
                sandbox: true, 
                   script: """
				     if (Environment.trim().toLowerCase().equals("others")){
						   inputBox="<input name='value' value='' type='text' style='width:100%;height:30%;background:white; color:black'>"
					 }else{
						 inputBox=""
					 }
				""".stripIndent()

            ]
        ]
    ]
	, [$class: 'DynamicReferenceParameter', 
        choiceType: 'ET_FORMATTED_HTML',
		omitValueField: true,
        description:'The username for connecting to the VCenter Server',
		randomName: 'string-parameter-02',
        name: 'VCenterUser',
        referencedParameters: 'Environment',
        script: [$class: 'GroovyScript',
            fallbackScript: [
                classpath: [], 
                sandbox: true, 
                script: """
						  inputBox="<input name='value' value='' type='text'>"
                """.stripIndent()
            ],
            script: [
                classpath: [], 
                sandbox: true, 
                   script: """
				     if (Environment.trim().toLowerCase().equals("others")){
						   inputBox="<input name='value' value='' type='text' style='width:100%;height:30%;background:white; color:black'>"
					 }else{
						 inputBox=""
					 }
				""".stripIndent()

            ]
        ]
    ]
	,[$class: 'DynamicReferenceParameter', 
        choiceType: 'ET_FORMATTED_HTML',
		omitValueField: true,
        description:'The password for connecting to the VCenter server',
		randomName: 'string-parameter-03',
        name: 'VCenterPassword',
        referencedParameters: 'Environment',
        script: [$class: 'GroovyScript',
            fallbackScript: [
                classpath: [], 
                sandbox: true, 
                script: """
						  inputBox="<input name='value' value='' type='password'>"
                """.stripIndent()
            ],
            script: [
                classpath: [], 
                sandbox: true, 
                   script: """
				     if (Environment.trim().toLowerCase().equals("others")){
						   inputBox="<input name='value' value='' type='password' style='width:100%;height:30%;background:white; color:black'>"
					 }else{
						 inputBox=""
					 }
				""".stripIndent()

            ]
        ]
    ]
    ,choice(name:'MicroManage',choices: ['YES','NO'], description:"Select 'Yes' to make changes to some additional/advanced Server configurations.")

    ])
 ])

pipeline{
    
    agent{
        
        docker{
            image 'vmware-powercli-tools'
            args  '-u 0'
            label 'linux'
        }
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
                    def  vmInformation       =  null 
                    if(params.Environment    in ['Test' ,'Production']){ 
                        vmInformation        =  getVMInfo(params.Environment)
                    }else{
                        vmInformation        =  getVMInfoOthers(params.VCenterServer,params.VCenterUser,params.VCenterPassword)
                    }
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

                    print(hostDatastoreMap)

                    
                    vmInformation['templates'].each{

                            vmTemplates.add(it.Name)

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

                    print(hostNetworkMap)

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
                    String vmName         =  params.ServerName.toString()
                    int vmCount           =  Integer.parseInt(params.ServerCount)
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

                    print("The value of MicroManage is ${params.MicroManage} ")
                    
                    if(params.MicroManage.toLowerCase()=="no"){ 
                        for( def k=0; k<vmCount; k++){
                                def natIndex = k+1
                                def vmSpecsMap = [:];
                                vmSpecsMap["name"]                  = natIndex.toString().length()==1?"${vmName}00${natIndex}":"${vmName}0${natIndex}"
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
                                updatedVMSpecifications = (vmSpecsMap);

                        }
                            
                    }else if(params.MicroManage.toLowerCase()=="yes"){ 

                        println("Preparing Server customization wizard")
                        
                    //  def vmHostMap = getVMHostMap(vmName,vmCount,physicalHosts)
                        
                    //  vmHostMap = readJSON (text :vmHostMap)
                    
                        for( def k=0; k<vmCount; k++){
                            def indexNatural = k+1
                            def vmIndex      = indexNatural
                            def vmSpecsMap   = [:]
                            def vmID         = "Server${indexNatural}"
                            def serverName   = indexNatural <9 ?"${vmName}00${indexNatural}":"${vmName}0${indexNatural}"
                            //def vmHostName = vmHostMap[vmID]

                        // echo "${vmHostName} has been chosen as the host of  ${serverName}"
                           
                            vmConfigInputParameters.addAll(
                                [
                                    separator(name: serverName, sectionHeader: "${serverName} Server Specifications",separatorStyle: "border-width: 0", sectionHeaderStyle: vmSeparatorHeaderStyle)
                                    ,string(name:"${serverName}_Name", defaultValue: serverName, description:'Choose a custom name for the new server')
                                    ,choice(name:"${serverName}_PhysicalHost", choices: physicalHosts, description: "Specify the physical host for ${serverName}")
                                    ,choice(name:"${serverName}_OS", choices: ['Windows','Linux'], description: "Specify the operating system for ${serverName}")
                                    ,choice(name:"${serverName}_VmTemplate", choices:vmTemplates , description: "Specify VM Template that would be used to create ${serverName}")
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
                                                        def index   = "${serverName}_PhysicalHost"
                                                       
                                                        return [${hostDatastoreMap['172.22.39.38']}]
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
                                            name: "${serverName}_NIC${index}"
                                            ,   referencedParameters: "${serverName}_PhysicalHost",
                                            script: [$class: 'GroovyScript',  
                                                        fallbackScript: [
                                                        classpath: [],  sandbox: true, 
                                                        script: """
                                                        return ["Fallback"]
                                                        """.stripIndent()
                                                        ] ,
                                                        script:[
                                                            classpath: [],   sandbox: true, 
                                                        classpath: [],   sandbox: true, 
                                                        script: """
                                                         def index   = ${serverName}_PhysicalHost
                                                        def options = ${hostNetworkMap}
                                                        return  ${hostNetworkMap}
                                                       """.stripIndent()
                                                        ]
                                                    ]
                                        ]
                                    
                                    ]
                                    )

                            }

        
                        } 
                      def vmSpecsModificationInput = input(id: 'vmSpecsModificationInput', message:'Click this link to provide additional information', parameters: vmConfigInputParameters, ok:'Provision')
                
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
