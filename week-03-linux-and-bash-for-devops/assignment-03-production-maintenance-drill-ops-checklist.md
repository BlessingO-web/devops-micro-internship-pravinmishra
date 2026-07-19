# Assignment 3 — Production Maintenance Drill (OPS Checklist)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will treat your already deployed React application (on Ubuntu VM with Nginx) as a live production system. You will perform structured operational checks covering network validation, service health, log analysis, resource monitoring, configuration verification, and incident simulation with recovery — mirroring real on-call DevOps responsibilities.

---

# Task 1 — Server Access & Networking Validation

## Goal

Verify that the deployed React application is reachable from the browser and confirm basic network connectivity of the Ubuntu VM.

### Evidence

#### Screenshot 1 — Browser showing the React app with your Full Name visible on the UI

![task1-Browser](screenshots/browser-react.png)

---

#### Screenshot 2 — Output of `ip a`

![task1-ipa](screenshots/2-ipa.png)

---

#### Screenshot 3 — Output of `sudo ss -tulpen`

![task1-tulpen](screenshots/3-tulpen.png)

---

#### Screenshot 4 — Output of `sudo ufw status`

![task1-ufw](screenshots/4-ufw.png)

---

### Notes

Answer the following in your own words:

**1. What proves Nginx is listening on 0.0.0.0:80?**

The output of sudo ss -tulpen showed Nginx listening on 0.0.0.0:80. This means Nginx is accepting HTTP requests from any network interface, allowing users to access the website through the browser.

---

**2. What proves SSH is active on port 22?**

The sudo ss -tulpen command showed SSH listening on port 22. I was also able to connect to the server using SSH, which confirms that the SSH service is running correctly.

---

**3. Did you find any unexpected open ports? Explain briefly.**

No. I only found the expected ports such as 22 for SSH and 80 for Nginx. This means the server is not exposing unnecessary services.

---

# Task 2 — Service Health & Systemd Validation (Nginx)

## Goal

Verify that Nginx is properly installed, running, enabled at boot, and safely configured.

### Evidence

#### Screenshot 1 — Output of `systemctl status nginx --no-pager`

![task2-nginx](screenshots/nginx-nopager.png)

---

#### Screenshot 2 — Output of `sudo nginx -t`

![task2-nginx-t](screenshots/nginx-t.png)

---

#### Screenshot 3 — Output of `sudo ss -lptn '( sport = :80 )'`

![task2-sudosport](screenshots/sudo-sport.png)

---

### Notes

Answer the following in your own words:

**1. What happens if Nginx fails to restart in production?**

If Nginx fails to restart, users will not be able to access the website because the web server will not be serving the application. This can lead to downtime until the issue is fixed.

---

**2. What's your basic rollback plan?**

I would restore the last working Nginx configuration, test it using nginx -t, and then restart the service. If the issue was caused by a deployment, I would also restore the previous application files.

---

# Task 3 — Logs & Request Trace

## Goal

Verify real traffic flow and analyze logs to understand system behavior and errors.

### Evidence

#### Screenshot 1 — Output of `sudo tail -n 30 /var/log/nginx/access.log`

![task3-access](screenshots/access.log.png)
![task3-access](screenshots/access.log1.png)

---

#### Screenshot 2 — Output of `sudo tail -n 30 /var/log/nginx/error.log`

![task3-error](screenshots/error.log.png)

---

#### Screenshot 3 — Output of `sudo journalctl -u nginx --no-pager -n 50`

![task3-journalctl](screenshots/journalctl-nginx.png)

---

### Notes

Answer the following in your own words:

**1. Were there any errors in the logs?**

- If yes, mention 1–2 example error lines from the logs and explain what each one means in simple terms.
- If no, explain what it means if the error log is empty or shows no recent errors during your check.

No. The Nginx error log did not show any recent errors during my checks. This means Nginx was able to handle requests successfully without any configuration or runtime issues.

---

**2. If there were no errors, what does that indicate about the system?**

It indicates that the web server was healthy and operating normally during the time I carried out the checks. The application was served successfully, and there were no internal errors affecting users.

---

**3. Based on the access logs, were your curl requests visible in the log entries? What does that prove about traffic flow?**

Yes. My curl requests appeared in the access log with a 200 OK response. This proves that the requests reached the Nginx server, were processed successfully, and the application responded correctly.

---

# Task 4 — System Resource Health Check (Capacity Red Flags)

## Goal

Assess server capacity and detect potential performance or failure risks.

### Evidence

#### Screenshot 1 — Output of `uptime`

![task4-uptime](screenshots/uptime.png)

---

#### Screenshot 2 — Output of `free -h`

![task4-free-h](screenshots/free-h.png)

---

#### Screenshot 3 — Output of `df -h`

![task4-df-h](screenshots/df-h.png)

---

#### Screenshot 4 — Output of `sudo du -sh /var/* | sort -h`

![task4-sort-h](screenshots/sort-h.png)

---

### Notes

Answer the following in your own words:

**1. Which resource looks most critical right now? (CPU/load, memory, or disk) Explain why.**

None of the resources were under pressure during my checks. However, I would pay closer attention to disk usage because it can gradually increase over time due to logs and application files. If it becomes full, it can affect the server's normal operation.

---

**2. What happens if disk becomes 100% full in a production server?**

If the disk becomes completely full, logs can no longer be written, applications may fail to save data, updates may stop working, and in severe cases the server can become unstable or even inaccessible.

---

# Task 5 — Configuration & Deployment Verification

## Goal

Ensure the correct React build is deployed and Nginx is serving it properly.

### Evidence

#### Screenshot 1 — Output of `ls -lah /var/www/html | head -n 20`

![task5-head-n](screenshots/head-n.png)

---

#### Screenshot 2 — Output of `grep -R "Deployed by" -n /var/www/html 2>/dev/null | head`

![task5-nullhead](screenshots/null-head.png)

---

#### Screenshot 3 — Output of `grep -n "try_files" /etc/nginx/sites-available/default`

![task5-try-file](screenshots/try-files.png)

---

### Notes

Answer the following in your own words:

**1. How do you confirm that the correct version of the application is deployed?**

I confirmed the correct deployment by checking the files in /var/www/html, verifying that the React build files were present, confirming my custom "Deployed by" text was included in the application, and ensuring Nginx was serving the application successfully.

---

# Task 6 — Nginx Configuration Failure Simulation

## Goal

Simulate a real-world Nginx misconfiguration and recover the service safely.

### Evidence

#### Screenshot 1 — Output of `sudo nginx -t` showing the syntax error (broken config)

![task6-broken](screenshots/broken-config.png)

---

#### Screenshot 2 — Output of `sudo nginx -t` showing syntax ok (fixed config)

![task6-nginx](screenshots/nginx-tsuccess.png)

---

#### Screenshot 3 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![task6-curl-i](screenshots/curl-i.png)

---

### Notes

Answer the following in your own words:

**1. What caused the configuration failure?**

The failure was caused by a syntax error in the Nginx configuration file after removing the required semicolon from the try_files directive. This made the configuration invalid.

---

**2. How did you fix the issue?**

I corrected the configuration by adding the missing semicolon, tested the configuration using sudo nginx -t, and restarted Nginx after confirming the configuration was valid.

---

**3. How can you avoid this kind of issue in real production systems?**

I would always test the Nginx configuration with nginx -t before restarting the service. I would also keep configuration files in version control, test changes in a staging environment, and automate configuration checks during deployment.

---

# Task 7 — Web Application Failure Simulation

## Goal

Simulate missing deployment content and recover the application safely.

### Evidence

#### Screenshot 1 — Output of `curl -I http://<public-ip>` showing failure (non-200 response)

![task7-backerror](screenshots/backup-error.png)

---

#### Screenshot 2 — Output of `curl -I http://<public-ip>` confirming recovery (200 OK)

![task7-recovery](screenshots/task7-recovery.png)

---

### Notes

Answer the following in your own words:

**1. What caused the application to break in this scenario?**

The application stopped working because the deployment files inside /var/www/html were removed. Although Nginx was still running, it could not find the React application files to serve to users.

---

**2. How did you fix the issue and restore the application?**

I restored the original application files from the backup directory, restarted the Nginx service, and confirmed the application was working again by checking the browser and verifying the server returned a 200 OK response.

---

**3. What steps would you take to prevent this kind of issue in real production systems?**

I would keep backups of every deployment, use version control, test deployments before releasing them, and perform health checks after deployment to quickly detect and recover from any issues.

---

# Task 8 — Security & Reliability Review

## Goal

Review and reflect on the security and reliability practices applied during this assignment.

### Security & Reliability Notes

Answer the following in your own words:

**1. Why is SSH key-based authentication more secure than sharing passwords?**

SSH key authentication is more secure because the keys are much harder to guess or crack than passwords. It also reduces the risk of password attacks and provides a safer way to access servers.

---

**2. Why should only required ports be open on a production server?**

Only the required ports should be open to reduce the server's attack surface. Closing unnecessary ports helps prevent unauthorized access and improves overall security.

---

**3. Why is it important for Nginx to be enabled on boot?**

Enabling Nginx on boot ensures that the web server starts automatically whenever the server restarts. This keeps the application available without requiring manual intervention.

---

**4. What are the risks of sharing secrets, keys, or credentials publicly?**

Sharing secrets or credentials publicly can allow unauthorized users to access cloud resources, steal sensitive data, modify infrastructure, or generate unexpected cloud costs.

---

**5. Why should cloud resources be stopped or terminated when they are no longer needed?**

Cloud resources should be stopped or terminated when they are no longer needed to avoid unnecessary charges, reduce security risks, and keep the cloud environment clean and well managed.

---

# LinkedIn Post (Required)

## Evidence

#### LinkedIn Post URL

Paste your LinkedIn post URL here:

`Add your URL here`

---

#### Screenshot — Published LinkedIn post

Add your screenshot here.

---

# Submission Instructions

- Add all required screenshots in your submission
- Full name must be visible in required screenshots
- Do not expose sensitive information (keys, passwords, account IDs)

---

# Completion Checklist

- [ ] Task 1: Screenshots (browser, ip a, ss -tulpen, ufw status) + Notes answered
- [ ] Task 2: Screenshots (nginx status, nginx -t, ss port 80) + Notes answered
- [ ] Task 3: Screenshots (access log, error log, journalctl) + Notes answered
- [ ] Task 4: Screenshots (uptime, free -h, df -h, du -sh) + Notes answered
- [ ] Task 5: Screenshots (ls html, grep deployed by, grep try_files) + Notes answered
- [ ] Task 6: Screenshots (nginx -t fail, nginx -t pass, curl recovery) + Notes answered
- [ ] Task 7: Screenshots (curl failure, curl recovery) + Notes answered
- [ ] Task 8: Security & Reliability Notes answered
- [ ] LinkedIn post published and URL submitted
- [ ] Full Name visible in all required screenshots
- [ ] No sensitive data exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*