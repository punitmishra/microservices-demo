def scmVars

properties([[$class: 'BuildDiscarderProperty',
             strategy: [$class: 'LogRotator', numToKeepStr: '10']]
])


podTemplate(label: 'microservice-demo-build', namespace: 'devops',
            nodeUsageMode: 'EXCLUSIVE',
            imagePullSecrets: [ 'callidus-open-source-gcr' ],
            containers: [
        containerTemplate(name: 'jnlp',
                          image: 'gcr.io/callidus-open-source/devops/jenkinsci/jnlp-slave:netcat',
                          ttyEnabled: true),
        containerTemplate(name: 'kaniko-nc-1',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          args: '44551',
                          alwaysPullImage: true,
                          ttyEnabled: true,
        ),
        containerTemplate(name: 'kaniko-nc-2',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          args: '44552',
                          ttyEnabled: true,
        ),
        containerTemplate(name: 'kaniko-nc-3',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          args: '44553',
                          ttyEnabled: true,
        ),
        containerTemplate(name: 'kaniko-nc-4',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          args: '44554',
                          ttyEnabled: true,
        ),
        containerTemplate(name: 'kaniko-nc-5',
                          image: 'gcr.io/callidus-open-source/devops/kaniko-nc:debug',
                          alwaysPullImage: true,
                          args: '44555',
                          ttyEnabled: true,
        )
    ],
            volumes: [
        secretVolume(mountPath: '/secret', secretName: 'callidus-opensource-kaniko-secret')
    ],
            envVars: [
        envVar(key: 'GOOGLE_APPLICATION_CREDENTIALS', value: '/secret/kaniko-secret.json')
    ])
{


    node('microservice-demo-build') {
        stage("Checkout") {
            scmVars = checkout scm
            def workspace = pwd()
            echo 'Workspace: ${workspace}'
        }

        stage("Build subprojects 1/2") {
            parallel(
                "build adservice": {
                    buildOnKanikoNc(workspace+"/src/adservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/adservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/adservice:${scmVars.GIT_COMMIT}"],
                                    44551)
                },
                "build cartservice": {
                    buildOnKanikoNc(workspace+"/src/cartservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/cartservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/cartservice:${scmVars.GIT_COMMIT}"],
                                    44552)
                },
                "build checkoutservice": {
                    buildOnKanikoNc(workspace+"/src/checkoutservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/checkoutservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/checkoutservice:${scmVars.GIT_COMMIT}"],
                                    44553)
                },
                "build currencyservice": {
                    buildOnKanikoNc(workspace+"/src/currencyservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/currencyservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/currencyservice:${scmVars.GIT_COMMIT}"],
                                    44554)
                },
                "build emailservice": {
                    buildOnKanikoNc(workspace+"/src/emailservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/emailservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/emailservice:${scmVars.GIT_COMMIT}"],
                                    44555)
                }
            )
        }

        stage("Build subprojects 2/2") {
            parallel(
                "build frontend": {
                    buildOnKanikoNc(workspace+"/src/frontend", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/frontend:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/frontend:${scmVars.GIT_COMMIT}"],
                                    44551)
                },
                "build paymentservice": {
                    buildOnKanikoNc(workspace+"/src/paymentservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/paymentservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/paymentservice:${scmVars.GIT_COMMIT}"],
                                    44552)
                },
                "build productcatalogservice": {
                    buildOnKanikoNc(workspace+"/src/productcatalogservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/productcatalogservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/productcatalogservice:${scmVars.GIT_COMMIT}"],
                                    44553)
                },
                "build recommendationservice": {
                    buildOnKanikoNc(workspace+"/src/recommendationservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/recommendationservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/recommendationservice:${scmVars.GIT_COMMIT}"],
                                    44554)
                },
                "build shippingservice": {
                    buildOnKanikoNc(workspace+"/src/shippingservice", "Dockerfile",
                                    ["gcr.io/callidus-open-source/microservice-demo/shippingservice:latest",
                                     "gcr.io/callidus-open-source/microservice-demo/shippingservice:${scmVars.GIT_COMMIT}"],
                                    44555)
                }
            )
        }

        stage("Build Load Generator") {
            buildOnKanikoNc(workspace+"/src/loadgenerator", "Dockerfile",
                            ["gcr.io/callidus-open-source/microservice-demo/loadgenerator:latest",
                             "gcr.io/callidus-open-source/microservice-demo/loadgenerator:${scmVars.GIT_COMMIT}"],
                            44551)
        }
    }

}



def buildOnKanikoNc(context, dockerfile, destination, ncport) {
    def destinations = ""
    destinations.each {dest ->
        destinations += "--destination=" + $dest + " "
    }

    echo "exit $(echo \"--dockerfile=${dockerfile} --context=${context} ${destinations} | nc 127.0.0.1 ${ncport} | tee /dev/fd/2 | grep -xoP \"^KANIKO_NC_RETURN=\\K\\d*\$\")"
    sh "exit $(echo \"--dockerfile=${dockerfile} --context=${context} ${destinations} | nc 127.0.0.1 ${ncport} | tee /dev/fd/2 | grep -xoP \"^KANIKO_NC_RETURN=\\K\\d*\$\")"
}
