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


#######################################################################################

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
