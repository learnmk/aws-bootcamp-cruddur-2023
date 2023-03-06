# Week 2 — Distributed Tracing

### TASK 1 
Instrumented backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider. 


Output of the queries run to explore traces within Honeycomb.io

![Screenshot 2023-03-06 at 4 25 52 PM](https://user-images.githubusercontent.com/125124581/223091310-6c65455c-5e96-4c7f-bca5-7391534f3bf4.png)


### TASK 2
Integrated AWS X-Ray into backend flask application. Configure and provisioned X-Ray daemon within docker-compose and send data back to X-Ray API. Observed X-Ray traces within the AWS Console. 

![Screenshot 2023-03-05 at 11 20 52 AM](https://user-images.githubusercontent.com/125124581/223090128-fc85296e-b1ac-419e-936c-2edca9a093fd.png)

![Screenshot 2023-03-05 at 11 59 24 AM](https://user-images.githubusercontent.com/125124581/223090233-c7569a81-7743-4b6a-9423-a9a75e1d219d.png)


### TASK 3
Install WatchTower and write a custom logger to send application log data to - CloudWatch Log group

![Screenshot 2023-03-05 at 1 12 29 PM](https://user-images.githubusercontent.com/125124581/223089987-91db726a-56d5-4921-88a8-eb6a38315fa6.png)


### TASK 4 
Integrate Rollbar for Error Logging. Trigger an error an observe an error with Rollbar


![Screenshot 2023-03-05 at 2 33 17 PM](https://user-images.githubusercontent.com/125124581/223089870-0d984174-c18a-4dbf-b235-e4b96d94eceb.png)
