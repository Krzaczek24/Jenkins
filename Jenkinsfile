import groovy.json.JsonSlurper
import groovy.json.JsonOutput

@NonCPS
def getSortedProjects(projects) {
    return projects.sort { it.order }
}

def printNiceHeader(str) {
	println "+${''.center(30, '-')}+"
	println "|${''.center(30, ' ')}|"
	println "+${''.center(30, '-')}+"
}

node {
	def currentPath = pwd()
	File file = new File("${currentPath}\\jenkinsParams.json")
	def slurper = new JsonSlurper()
	def jsonText = file.getText()
	def params = slurper.parseText(jsonText)
	
	printNiceHeader('Loaded JSON')
	println JsonOutput.prettyPrint(jsonText)

	def pathTemplate = params.pathTemplates.default
	def sortedProjectNames = getSortedProjects(params.projects)
	def allProjectNames = sortedProjectNames*.name
	def gitPath = params.gitPath
	def dotNetPath = params.dotNetPath

	def allProjectsData = []
	def selectedProjects = Projects.tokenize(',')

	for (projectName in allProjectNames) {
		allProjectsData.add([name: projectName, path: pathTemplate.replace("__project__", projectName)])
	}

	
	println "################"
    checkout scm
	println "################"
    
    stage('Checkout') {
        git url: gitPath, branch: 'CICD'
    }
    
    stage('Restore packages') {
        if (Restore == true) {
            for (project in allProjectsData) {
				if (selectedProjects.contains(project)) {
					println "Restoring packages for '${project.name}' project ... "
					bat "${dotNetPath} restore ${project.path}"
				}
				else {
					println "Skipped restoring packages for '${project.name}' project"
				}
            }
        }
		else {
			println "Skipped restoring packages for all projects"
		}
    }
    
    stage('Clean') {
        if (Clean == true) {
	        for (project in allProjectsData) {
				if (selectedProjects.contains(project)) {
					println "Cleaning for '${project.name}' project ... "
					bat "${dotNetPath} clean ${project.path}"
				}
				else {
					println "Skipped cleaning '${project.name}' project"
				}
            }   
	    }
		else {
			println "Skipped cleaning all projects"
		}
    }
    
	for (project in allProjectsData) {
		stage("Build '${project.name}'") {
			if (selectedProjects.contains(project)) {
				println "Building '${project.name}' project ... "
				bat "${dotNetPath} build ${project.path} --configuration Release"
			}
			else {
				println "Skipped building '${project.name}' project"
			}
		}
	}
	
    stage('Unit Test') {
        println "No unit tests definied"
		//bat "${dotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
    
    stage('Integration Test') {
        println "No unit tests definied"
		//bat "${dotNetPath} test C:\\Windows\\System32\\config\\systemprofile\\AppData\\Local\\Jenkins\\.jenkins\\jobs\\DynamicManager\\workspace\\DynamicManager\\Test\\DynamicManager.Test.csproj"
    }
	
	for (project in allProjectsData) {
		stage("Publish '${project.name}'") {
			if (selectedProjects.contains(project)) {
				println "Publishing '${project.name}' project ... "
				bat "${dotNetPath} publish ${project.path}"
			}
			else {
				println "Skipped publishing '${project.name}' project"
			}
		}
	}
}
