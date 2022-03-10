pipeline{
 agent {
   node {
     label 'maven'
   }
 }
 environment {
    RHT_OCP4_DEV_USER = 'dggzdv'
    ENVIRONMENT='production'
    DEPLOYMENT_CONFIG_PRODUCTION = 'shopping-cart-production'
 }
 stages {
   stage('Test') {
     steps {
       sh './mvnw clean test'
     }
   }
   stage('Package') {
      steps {
        sh './mvnw clean package -DskipTests -Dquarkus.package.type=uber-jar'
        archiveArtifacts 'target/*.jar'
      }
   }
   stage('Build Image') {
     environment { MY_QUAY = credentials('DO400_QUAY_USER') }
     steps {
        sh './mvnw quarkus:add-extension -Dextensions="kubernetes,container-image-jib"'
        sh '''
            ./mvnw clean package -DskipTests \
            -Dquarkus.container-image.build=true \
            -Dquarkus.container-image.registry=quay.io \
            -Dquarkus.container-image.group=$MY_QUAY_USR \
            -Dquarkus.container-image.name=do400-deploying-environments \
            -Dquarkus.container-image.username=$MY_QUAY_USR \
            -Dquarkus.container-image.password="$MY_QUAY_PSW" \
            -Dquarkus.container-image.push=true
        '''
     }
   }
   stage('Template - Prepate') {
    environment { MY_QUAY = credentials('DO400_QUAY_USER') }
      steps{
        sh "oc new-project ${RHT_OCP4_DEV_USER}-${DEPLOYMENT_CONFIG_PRODUCTION}"
        sh "oc process -n ${RHT_OCP4_DEV_USER}-${DEPLOYMENT_CONFIG_PRODUCTION} -f ./kubefiles/application-template.yml -p QUAY_USER_OR_GROUP=$MY_QUAY_USR -p APP_ENVIRONMENT=${ENVIRONMENT} > ./kubefiles/${DEPLOYMENT_CONFIG_PRODUCTION}.yml "
        sh "oc apply -n ${RHT_OCP4_DEV_USER}-${DEPLOYMENT_CONFIG_PRODUCTION} -f ./kubefiles/${DEPLOYMENT_CONFIG_PRODUCTION}.yml"
        sh "oc policy add-role-to-user edit system:serviceaccount:${RHT_OCP4_DEV_USER}-jenkins:jenkins"
      }
    }
   stage('Deploy - Production Env') {
      environment {
        APP_NAMESPACE = "${RHT_OCP4_DEV_USER}-${DEPLOYMENT_CONFIG_PRODUCTION}"
      }
      input { message 'Deploy to Production Environment?' }
      steps{
        sh "oc rollout latest dc/${DEPLOYMENT_CONFIG_PRODUCTION} -n ${APP_NAMESPACE}"
      }
     }
 }
}
