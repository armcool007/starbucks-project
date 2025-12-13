Aapne Gmail SMTP configure kar liya hai Jenkins mein, lekin Email Extension Plugin (Email-ext) install karna important kyun hai—ye samajhne ke liye pehle dono cheezon ka role alag se samajhte hain:

🔹 1. Built-in Email Notification (Jenkins ki default email setting)
Jenkins ke "Configure System" mein SMTP settings (jaise Gmail SMTP) configure karne se aap basic email notifications bhej sakte hain.
Ye sirf "Email Notification" post-build action ke through kaam karta hai.
Limitations:
Fixed content (jaise build status, console log link).
Custom subject, body, triggers (jaise only on failure) itne flexible nahi hote.
Recipients static hote hain (global ya project-level).
🔹 2. Email Extension Plugin (Email-ext) – Advanced Email Capability
Ye plugin aapko powerful, customizable email notifications deta hai:

✅ Kyun important hai?
Custom Triggers:
Sirf "failure" pe nahi, balki unstable, success, first failure, fixed, etc. pe alag-alag emails bhej sakte ho.
Dynamic Content:
HTML/Text email templates banaye ja sakte hain.
Jenkins environment variables (jaise ${BUILD_NUMBER}, ${GIT_COMMIT}) use karke dynamic info dikhaye ja sakte hain.
Attachments & Logs:
Build console output, test reports, ya koi file email ke sath attach kar sakte ho.
Recipient Management:
Committer ko email bhejo, ya specific user groups ko—flexible recipient list.
Pre-send Script Support:
Groovy script se email content ya recipients dynamically decide kar sakte ho.


######################################################################################################################################################################
environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
1️⃣ environment { }
Ye block environment variables define karne ke liye hota hai
Ye variables poori pipeline ke sab stages me available hote hain
2️⃣ SCANNER_HOME
Ye variable ka naam hai
Tum koi bhi naam rakh sakte ho (mostly capital letters use karte hain)
📌 Example:
SCANNER_HOME=/opt/sonar-scanner
3️⃣ tool 'sonar-scanner'
tool Jenkins ka built-in function hai
Ye Jenkins ke Global Tool Configuration se tool ka path nikalta hai
👉 'sonar-scanner' tool ka exact name hona chahiye
(Jenkins → Manage Jenkins → Global Tool Configuration)
######################################################################################################################################################################

                                                            "cleanWs()" ka bahut hi simple matlab:

✅ Jenkins workspace ko poora clean (saaf) kar dena
Yani:

Purane build ke files
Temporary files
Logs
Artifacts
Hidden files
Koi bhi bacha hua code
sab delete ho jata hai.
Workspace bilkul fresh ho jata hai, jaise naya build start ho raha

####################################################################################################################################################################

stage("Sonarqube Analysis "){ 
  steps{ 
    withSonarQubeEnv('SonarQube') {
      sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=starbucks \ -Dsonar.projectKey=starbucks ''' 
    } 
  } 
}

✅ -D = Define (property set karna)
  -Dsonar.someProperty=value = SonarQube ko koi setting dena.
    -D → Define property
    sonar.xxx → SonarQube ki property

✅ 1. stage("Sonarqube Analysis ")
Iska matlab:
👉 “Ab hum SonarQube se code ka analysis (scan) karenge.”

✅ 2. withSonarQubeEnv('SonarQube')
Iska matlab:
👉 “SonarQube ke server ka environment use karo.”
Jenkins me jo SonarQube naam ka server configure hai,
uske URL, token, login details sab is block ke andar available ho jaate hain.

✅ 3. sh ''' $SCANNER_HOME/bin/sonar-scanner ...
Iska matlab:
👉 “Sonar-scanner command chalao.”
Yeh command code scan karta hai aur report SonarQube server ko bhej deta hai.
🟢 4. Scanner command ke options:
🔸 -Dsonar.projectName=starbucks
👉 SonarQube me project ka naam “starbucks” rakho.
🔸 -Dsonar.projectKey=starbucks
👉 Project ka unique key “starbucks” ho.
(project key = SonarQube me ek unique ID)

$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$
stage("quality gate"){
  steps {
    script {
      waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
    } 
  }
}
✅ 1. waitForQualityGate
Ye Jenkins ko bolta hai:
“SonarQube ka final verdict (Pass/Fail) aane tak ruk jao.”
Code likhne se pehle SonarQube scan ho chuka hota hai.
Jab tak SonarQube result nahi deta → Jenkins yahan rukta hai.

✅ 2. abortPipeline: false ka matlab
Agar abortPipeline: true hota:
➡️ Agar SonarQube Quality Gate Fail → Puri pipeline ko turant STOP kar deta.
But tumne likha hai:
abortPipeline: false
➡️ Matlab:
Quality Gate fail hone par bhi pipeline FAIL nahi karega — pipeline aage chalegi.

✅ 3. credentialsId: 'Sonar-token' ka matlab
Jenkins me tumne ek secret token store kiya hoga named:
Sonar-token
Ye token Jenkins ko SonarQube se baat karne deta hai.
Without token → Jenkins SonarQube se result fetch nahi kar sakta.

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Trivy ko hamesha Jenkins machine (Jenkins agent) par install hona chahiye.
SonarQube machine par Trivy install karne ki bilkul zaroorat nahi hai.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

✔ post { always { ... }}
Matlab pipeline ke last me hamesha yeh part chalega
(fail ho ya success → fark nahi padhta)
def buildStatus = currentBuild.currentResult

✔ buildStatus
Yeh current build ka result store karta hai:
SUCCESS
FAILURE
UNSTABLE
ABORTED
def buildUser = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')[0]?.userId ?: 'Github User'

✔ buildUser
Yeh find karta hai kisne build start kiya
Manual run → Jenkins user ka naam
GitHub push → "Github User"
?: 'Github User' matlab:
Agar user nahi mila → default "Github User"
📩 EMAIL SENDING PART
emailext (
    subject: "Pipeline ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
✔ Email subject
Example:
Pipeline SUCCESS: starbucks-pipeline #12

body: """
    <p>This is a Jenkins starbucks CICD pipeline status.</p>
    <p>Project: ${env.JOB_NAME}</p>
    <p>Build Number: ${env.BUILD_NUMBER}</p>
    <p>Build Status: ${buildStatus}</p>
    <p>Started by: ${buildUser}</p>
    <p>Build URL: <a href="${env.BUILD_URL}">${env.BUILD_URL}</a></p>
""",

✔ Email body (HTML format)
Email me yeh details jayegi:
Project name
Build number
Build status
Kisne start kiya
Build URL (clickable)
to: 'mohdaseemakram19@gmail.com',
from: 'mohdaseemakram19@gmail.com',
replyTo: 'mohdaseemakram19@gmail.com',
mimeType: 'text/html',

✔ Receiver / sender email config
to: → jisko email jaayegi
from: → kaun bhej raha
replyTo: → reply kahan aaye
mimeType: text/html → HTML formatting allow (bold, colors etc.)
attachmentsPattern: 'trivyfs.txt,trivyimage.txt'

✔ Email ke sath kaunse files attach honge
Trivy file-system scan report
Trivy image scan report
Email ke sath PDFs ki tarah attach ho jayenge.
