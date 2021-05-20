pipeline {
agent none
    
stages {
stage('Downloading code from GIT'){
steps{
git branch: 'FY21-Release4beforeR5', credentialsId: '59b26bb8-157c-4538-9b1c-9c7b4af06c0f', poll: false, url: 'http://10.100.8.119:7990/scm/hpjuke/hp2b.git'
}
}

stage('Artifactory Upload Details'){
steps{
rtUpload (
    serverId: 'http://10.100.8.226:8081/artifactory',
    spec: '''{
          "files": [
            {
            "pattern": "**/wcbd-deploy-server-$BUILD_NUMBER.zip",
            "target": "Jukebox_WCS_HP2B_FY20-Release2"
        },
        {
            "pattern": "**/wcbd-deploy-search-$BUILD_NUMBER.zip",
            "target": "Jukebox_WCS_HP2B_FY20-Release2"
        },       
        {
            "pattern": "buildtag-$BUILD_NUMBER.*",
            "target": "Jukebox_WCS_HP2B_FY20-Release2"
        }
         ]
    }''',
)
}
}

stage('Capturing Build Data'){
steps{
rtBuildInfo (
    captureEnv: true,
    excludeEnvPatterns: ['*password*', '*secret*', '*key*'],
)
}
}
		
stage('Building the Package'){
steps {
sh '''
mkdir -p -m775 target server
cd ${WORKSPACE}/source/workspace/Stores/WebContent/JukeboxSAS
ln -s ~/node_modules node_modules
gulp minify-js
gulp minify-css
rm node_modules
cd /opt/IBM/WebSphere/CommerceServer80/wcbd

./wcbd-ant -buildfile wcbd-nopackage-build.xml -Dbuild.label=$BUILD_NUMBER -Ddist.dir=${WORKSPACE}/target/app \
-Ddist.server.dir=${WORKSPACE}/target/app -Dbuild.branch=FY20-Release2 \
-Dworking.package.server.dir=${WORKSPACE}/server/app \
-Dsource.dir=${WORKSPACE}/source \
-Drun.extract=false

echo 
echo

cd search
pwd
./wcbd-ant -buildfile wcbd-build.xml -Dbuild.label=$BUILD_NUMBER -Ddist.dir=${WORKSPACE}/target/search \
-Ddist.server.dir=${WORKSPACE}/target/search -Dbuild.branch=FY20-Release2 \
-Dworking.package.server.dir=${WORKSPACE}/server/search \
-Dsource.dir=${WORKSPACE}/source \
-Drun.extract=false

echo "BUILD_JOB_TAG=$BUILD_TAG" > ${WORKSPACE}/buildtag.properties
echo "$BUILD_TAG" > ${WORKSPACE}/buildtag.html

cd ${WORKSPACE}
cp buildtag.properties buildtag-$BUILD_NUMBER.properties
cp buildtag.html buildtag-$BUILD_NUMBER.html
'''
}
}

stage('Pushing Tag Details to Git'){
steps{
job('example-2') {
    publishers {
        git {
            pushOnlyIfSuccess()
            tag('$BUILD_TAG', 'foo-$PIPELINE_VERSION') {
                message('Release $PIPELINE_VERSION')
                create()
            }
        }
    }
}
