import groovy.json.JsonSlurper

node {
	def currentPath = pwd()
	File file = new File("${currentPath}\\jenkinsParams.json")
	def slurper = new JsonSlurper()
	def jsonText = file.getText()
	def json = slurper.parseText(jsonText)

	def sortedProjectNames = json.projects.sort { it.order }
	def pathTemplate = json.pathTemplates.default
	def allProjectNames = sortedProjectNames*.name
	def gitPath = json.gitPath
	def dotNetPath = json.dotNetPath

	def allProjectsData = []
	def selectedProjects = Projects.tokenize(',')

	for (projectName in allProjectNames) {
		projectsData.add([name: projectName, path: pathTemplate.replace("__project__", projectName)])
	}

    checkout scm
    
    stage('Checkout') {
        git url: gitPath, branch: 'CICD'
    }
    
    stage('Restore packages') {
        if (Restore == true) {
            for (project in projectsData) {
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
	        for (project in projectsData) {
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
    
	for (project in projectsData) {
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
	
	for (project in projectsData) {
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
