#!/usr/bin/groovy

def buildManifest = {String manifest, String bitbake_image ->
    // Store the directory we are executed in as our workspace.
    String workspace = pwd()

    try {
        // Stages are subtasks that will be shown as subsections of the finiished build in Jenkins.

        stage('Download') {
            // Checkout the git repository and refspec pointed to by jenkins
            checkout scm
            // Update the submodules in the repository.
            sh 'git submodule update --init'
        }

        stage('Start Vagrant') {
            // Calculate available amount of RAM
            String gigsramStr = sh (
                script: 'free -tg | tail -n1 | awk \'{ print $2 }\'',
                returnStdout: true
            )
            int gigsram = gigsramStr.trim() as Integer
            // Cap memory usage at 8GB
            if (gigsram >= 8) {
                gigsram = 8
                println "Will set VAGRANT_RAM to ${gigsram}"
            }

            // Start the machine (destroy it if present) and provision it
            sh "cd ${workspace}"
            sh "vagrant destroy -f || true"
            withEnv(["VAGRANT_RAM=${gigsram}"]) {
                sh "vagrant up"
            }
        }

        stage('Repo init') {
            sh "vagrant ssh -c \"/vagrant/ci_scripts/do_repo_init ${manifest}\""
        }

        stage('Setup TEMPLATECONF') {
            String bsp = manifest.split('-')[1].split(".")[0]
            withEnv(["TEMPLATECONF" => "/home/vagrant/pelux_yocto/sources/meta-pelux-bsp-${bsp}/conf") {
                sh "vagrant ssh -c \"/vagrant/vagrant-cookbook/yocto/fetch-sources-for-recipes.sh\""
            }
        }

        stage('Bitbake image') {
            sh "vagrant ssh -c \"/vagrant/vagrant-cookbook/yocto/build-images.sh ${bitbake_image}\""
        }
    }

    catch(err) {
        // Do not add a stage here.
        // When "stage" commands are run in a different order than the previous run
        // the history is hidden since the rendering plugin assumes that the system has changed and
        // that the old runs are irrelevant. As such adding a stage at this point will trigger a
        // "change of the system" each time a run fails.
        println "Something went wrong!"
        currentBuild.result = "FAILURE"
    }

    // Always try to shut down the machine
    // Shutdown the machine
    sh "vagrant destroy -f || true"
}

node {
    buildManifest("pelux-intel.xml", "core-image-pelux")
}

node {
    buildManifest("pelux-intel-qt.xml", "core-image-pelux-qt")
}
