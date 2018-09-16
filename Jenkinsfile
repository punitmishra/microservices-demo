def scmVars

properties([[$class: 'BuildDiscarderProperty',
             strategy: [$class: 'LogRotator', numToKeepStr: '10']]
])

def buildOnKanikoNc(context, dockerfile, destination, ncport) {
    def destinations = ""
    destinations.each {dest ->
        destinations += "--destination=" + $dest + " "
    }

    echo 'exit $(echo "--dockerfile=${dockerfile} --context=${context} ${destinations} | nc 127.0.0.1 ${ncport} | tee /dev/fd/2 | grep -xoP "^KANIKO_NC_RETURN=\\K\\d*\$")'
    sh 'exit $(echo "--dockerfile=${dockerfile} --context=${context} ${destinations} | nc 127.0.0.1 ${ncport} | tee /dev/fd/2 | grep -xoP "^KANIKO_NC_RETURN=\\K\\d*\$")'
}

podTemplate(label: 'microservice-demo-build', namespace: 'devops',
            nodeUsageMode: 'EXCLUSIVE',
            imagePullSecrets: [ 'callidus-open-source-gcr' ],
            containers: [
        containerTemplate(name: 'jnlp',
                          image: 'gcr.io/callidus-open-source/devops/jenkinsci/jnlp-slave:netcat',
                          ttyEnabled: true),
        containerTemplate(name: 'kaniko-nc-1',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          ttyEnabled: true,
                          ports: [portMapping(name: 'nc', containerPort: 44556, hostPort: 44551)]
        ),
        containerTemplate(name: 'kaniko-nc-2',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          ttyEnabled: true,
                          ports: [portMapping(name: 'nc', containerPort: 44556, hostPort: 44552)]
        ),
        containerTemplate(name: 'kaniko-nc-3',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          ttyEnabled: true,
                          ports: [portMapping(name: 'nc', containerPort: 44556, hostPort: 44553)]
        ),
        containerTemplate(name: 'kaniko-nc-4',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          ttyEnabled: true,
                          ports: [portMapping(name: 'nc', containerPort: 44556, hostPort: 44554)]
        ),
        containerTemplate(name: 'kaniko-nc-5',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          ttyEnabled: true,
                          ports: [portMapping(name: 'nc', containerPort: 44556, hostPort: 44555)]
        )
    ],
            volumes: [
        secretVolume(mountPath: '/secret', secretName: 'callidus-opensource-kaniko-secret')
    ],
            envVars: [
        envVar(key: 'GOOGLE_APPLICATION_CREDENTIALS', value: '/secret/kaniko-secret.json')
    ])
{

    node('kaniko-build') {
        stage("Checkout") {
            scmVars = checkout scm
            def workspace = pwd()
            echo 'Workspace: ${workspace}'
        }

        stage("Build subprojects 1/2") {
            parallel(
                "build dss_design": {
                    buildOnKanikoNc(workspace+"/common/", "dss_design.df",
                                    ["gcr.io/ace-element-175818/thunderbridge-ai/dss_design::latest",
                                     "gcr.io/ace-element-175818/thunderbridge-ai/dss_design:latest:${scmVars.GIT_COMMIT}"],
                                    44551)
                },
                "build dss_score": {
                    buildOnKanikoNc(workspace+"/commom", "dss_score.df",
                                    ["gcr.io/ace-element-175818/thunderbridge-ai/dss_score:latest",
                                     "gcr.io/ace-element-175818/thunderbridge-ai/dss_score:${scmVars.GIT_COMMIT}"],
                                    44552)
                })
              }
          }
    }
