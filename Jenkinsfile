import groovy.json.JsonSlurper
import groovy.json.JsonOutput

@NonCPS
def getSortedProjects(projects) {
    return projects.sort { it.order }
}

@NonCPS
def getProjectsToReRun(projects) {
	return projects.findAll { it.rerun == true }
}

def printNiceHeader(str) {
	println "+${''.center(128, '-')}+\n|${''.center(8, ' ')}${str.padRight(120, ' ')}|\n+${''.center(128, '-')}+"
}

node {
	printNiceHeader("Checkout SCM")
	checkout scm
	
	def allProjectsData = []
	def selectedProjects = Projects.tokenize(',')
	def processesToStop = []
	def executablesToRun = []
    
    stage('Checkout') {
		printNiceHeader("Checkout ${GitRepoPath}")
        git url: GitRepoPath, branch: 'CICD'
		
		def currentPath = pwd()
		File file = new File("${currentPath}\\jenkinsParams.json")
		def slurper = new JsonSlurper()
		def jsonText = file.getText()
		def params = slurper.parseText(jsonText)
		
		printNiceHeader('Loaded "jenkinsParams.json"')
		println JsonOutput.prettyPrint(jsonText)

		def pathTemplate = params.pathTemplates.default
		def sortedProjectNames = getSortedProjects(params.projects)
		def allProjectNames = sortedProjectNames*.name

		def printProjects = 'PROJECTS:\n'
		for (projectName in allProjectNames) {
			def projectData = params.projects.find {proj -> proj.name == projectName}
			
			def csprojPath = params.pathTemplates[projectData.projectPathTemplate].replace("__project__", projectName)
			def exePath = params.pathTemplates[projectData.executablePathTemplate].replace("__project__", projectName)
			def processName = params.pathTemplates[projectData.processNameTemplate].replace("__project__", projectName)
			def rerun = projectData.toRestart
			
			allProjectsData.add([name: projectName, csproj: csprojPath, exe: exePath, process: processName, rerun: rerun])
			printProjects += "${(selectedProjects.contains(projectName) ? '+' : '-')} ${projectName}\n"
		}
	    
	    	def toReRunProjects = getProjectsToReRun(allProjectsData)
	    
	    for (project in toReRunProjects) {
		    processesToStop.add(project.process)
		    executablesToRun.add(project.exe)
	    }
		
		printNiceHeader("Parameters")
		println "${printProjects}ACTIONS:\n${Restore == 'true' ? '+' : '-'} Restore\n${Clean == 'true' ? '+' : '-'} Clean"
    }
    
    stage('Restore packages') {
		printNiceHeader("Restore packages")
        if (Restore == 'true') {
            for (project in allProjectsData) {
				if (selectedProjects.contains(project.name)) {
					println "Restoring packages for '${project.name}' project ... "
					bat "${DotNetPath} restore ${project.path}"
				} else {
					println "Skipped restoring packages for '${project.name}' project"
				}
            }
        } else {
			println "Skipped restoring packages for all projects"
		}
    }
    
    stage('Clean') {
		printNiceHeader("Clean")
        if (Clean == 'true') {
	        for (project in allProjectsData) {
				if (selectedProjects.contains(project.name)) {
					println "Cleaning for '${project.name}' project ... "
					bat "${DotNetPath} clean ${project.path}"
				} else {
					println "Skipped cleaning '${project.name}' project"
				}
            }   
	    } else {
			println "Skipped cleaning all projects"
		}
    }
    
	printNiceHeader("Build")
	for (project in allProjectsData) {
		stage("Build '${project.name}'") {
			if (selectedProjects.contains(project.name)) {
				println "Building '${project.name}' project ... "
				bat "${DotNetPath} build ${project.path} --configuration Release"
			} else {
				println "Skipped building '${project.name}' project"
			}
		}
	}
	
    stage('Unit Test') {
		printNiceHeader("Unit Test")
        println "No unit tests definied"
		//bat "${DotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
    
    stage('Integration Test') {
		printNiceHeader("Integration Test")
        println "No unit tests definied"
		//bat "${DotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
	printNiceHeader("Publish")
	println "Stoping processes:"
	for (process in processesToStop) {
		println process
		bat "wmic processwhere name=\"${process}\" delete"
	}
	
	for (project in allProjectsData) {
		stage("Publish '${project.name}'") {
			if (selectedProjects.contains(project.name)) {
				println "Publishing '${project.name}' project ... "
				bat "${DotNetPath} publish -c Release ${project.path}"
			} else {
				println "Skipped publishing '${project.name}' project"
			}
		}
	}
	
	println "Running executables:"
	for (exe in executablesToRun) {
		println exe
		bat "${exe}"
	}
}
