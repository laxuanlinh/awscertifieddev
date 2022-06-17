**Auto Scaling Group**

- Target tracking scaling: ex ASG CPU to stay around 40%
- Simple/Step scaling: ex when CloudWatch alarm is triggered (>70%), add 2 instances, when (<30%) remove 1 instance
- Scheduled action: ex increase min capacity at 10:00AM today
- Predictive scaling: forecast and schedule scaling ahead based on machine learning

**Some good metrics to scale:**

- CPU utilization
- Request per target
- Avg network in/out
- Any custom metric

**Scaling cooldowns:**

- When there is a scaling activiy happens, there is a cooldown period (default 300s) that the ASG does not launch or terminate any instances.
- Best to use ready-to-use AMI to reduce config time to serve requests faster and reduce cooldown time.
