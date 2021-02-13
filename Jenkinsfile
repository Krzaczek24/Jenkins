import groovy.json.JsonSlurper
import groovy.json.JsonOutput

@NonCPS
def getSortedProjects(projects) {
    return projects.sort { it.order }
}

def printNiceHeader(str) {
	println "+${''.center(128, '-')}+\n|${''.center(8, ' ')}${str.padRight(120, ' ')}|\n+${''.center(128, '-')}+"
}

node {
	printNiceHeader("Checkout SCM")
	checkout scm
	
	def allProjectsData = []
	def selectedProjects = Projects.tokenize(',')
    
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
			allProjectsData.add([name: projectName, path: pathTemplate.replace("__project__", projectName)])
			printProjects += "${(selectedProjects.contains(projectName) ? '+' : '-')} ${projectName}\n"
		}
		
		printNiceHeader("Parameters")
		println "${printProjects}ACTIONS:\${Restore ? '+' : '-'} Restore\n${Clean ? '+' : '-'} Clean"
    }
    
    stage('Restore packages') {
		printNiceHeader("Restore packages")
        if (Restore == true) {
            for (project in allProjectsData) {
				if (selectedProjects.contains(project)) {
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
        if (Clean == true) {
	        for (project in allProjectsData) {
				if (selectedProjects.contains(project)) {
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
			if (selectedProjects.contains(project)) {
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
	for (project in allProjectsData) {
		stage("Publish '${project.name}'") {
			if (selectedProjects.contains(project)) {
				println "Publishing '${project.name}' project ... "
				bat "${DotNetPath} publish ${project.path}"
			} else {
				println "Skipped publishing '${project.name}' project"
			}
		}
	}
}
