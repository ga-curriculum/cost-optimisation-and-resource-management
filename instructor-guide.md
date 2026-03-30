# Instructor Guide: Cost Optimisation and Resource Management

## Module Overview
In this module, instructors will guide learners to analyse how to optimise resource allocation and cloud costs without sacrificing performance. The focus is on the financial realities of running AI at scale. Learners will learn to monitor usage patterns and apply automation for efficiency.

## Learning Objectives
By the end of this session, learners will be able to:
* Identify cost drivers in AI infrastructure management.
* Apply monitoring and automation techniques to reduce unnecessary spend.
* Evaluate the balance between performance, scalability, and cost efficiency.

## Session Pacing Guide (120 Minutes total)

| Duration   | Segment                          | Instructor Action                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|:-----------|:---------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **20 min** | **Identifying Cost Drivers**     | Discuss the hidden costs of AI: Idle GPUs, data egress fees, and over-provisioned memory. Draw the "FinOps Triangle" (Speed, Cost, Quality) on the board.                                                                                                                                                                                                                                                                                          |
| **40 min** | **Automation & Trade-offs Demo** | Live demonstration: Show a simple Apache Airflow DAG triggering a batch inference job in Python. Discuss how scheduling jobs during off-peak hours reduces costs. Airflow must be running before the session starts; start the webserver and scheduler 10 minutes before the demo. Note that initial setup requires `pip install apache-airflow` (~5 minutes), running `airflow db migrate` (~1 minute), and creating an admin user (~30 seconds). |
| **60 min** | **Lab: Workflow Automation Lab** | Facilitate the lab where learners build a lightweight, event-driven workflow using Apache Airflow. This lab reinforces automation principles for managing data pipelines efficiently.                                                                                                                                                                                                                                                              |

---

## Delivery Notes & Teaching Tips

### 1. The "Always On" Fallacy
* **Simplifying Concepts:** Many engineers treat AI models like standard web servers, keeping them running 24/7. Explain that AI accelerators are too expensive for this. Introduce "Scale-to-Zero" and event-driven architectures where the infrastructure only spins up when a request is made or a batch job is scheduled.
  
* **Learner Context:** Frame this module around ROI (Return on Investment). Show them that saving thousands of dollars a month on cloud bills makes them highly valuable to their business stakeholders.

### 2. Troubleshooting Airflow DAG Visibility
Students are almost guaranteed to encounter issues with their DAGs not appearing in the Airflow UI during the lab. Provide this crucial troubleshooting tip upfront:
* Instruct learners to run `python dags/my_dag.py` directly in the terminal to catch syntax errors.
* Remind them to verify that the Airflow configuration is pointing to the correct DAGs folder path.

---

## Common Student Issues & Solutions

| Issue                                              | Potential Cause                                                                                                 | Solution                                                                                                                                                                                                        |
|:---------------------------------------------------|:----------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Airflow Webserver Fails to Start**               | Port 8080 is already in use by another local application, or Python dependencies are missing.                   | Instruct the student to find and kill the blocking process on port 8080, or change the Airflow port via environment variables. Ensure they activated their Python virtual environment.                          |
| **DAGs are not appearing in the UI**               | The Airflow configuration is pointing to the wrong folder, or there is a Python syntax error in their DAG file. | Have the learner run `python dags/my_dag.py` directly in the terminal to catch syntax errors. If it runs cleanly, verify the DAGs folder path.                                                                  |
| **Task is stuck in "Queued" state**                | The Airflow Scheduler process is not running in the background.                                                 | Remind learners that Airflow requires *both* the webserver and the scheduler to run simultaneously. Open a new terminal tab and run `airflow scheduler`.                                                        |
| **Confusion over "Spot Instances" vs "On-Demand"** | Applying traditional high-availability web hosting mentalities to asynchronous AI workloads.                    | Clarify that Spot Instances can be terminated by the cloud provider at any time. They are perfect for resumable batch training (which Airflow can easily restart) but terrible for real-time customer chatbots. |

---

## Debrief Questions
1. "In the Workflow Automation Lab, how did building a lightweight, event-driven workflow using Apache Airflow reinforce automation principles for managing data pipelines efficiently?"
2. "If an AI model's accuracy drops by 2% when using a smaller, cheaper quantization method, how would you evaluate if that trade-off is worth a 50% reduction in cloud costs?"
3. "What are the hidden costs associated with moving large training datasets between different cloud availability zones to take advantage of cheaper compute?"

---

## Resources for the Instructor
* **Reference:** 
  * [Apache Airflow Core Concepts](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/dags.html)
  * [FinOps Foundation: Cloud Financial Management](https://www.finops.org/introduction/what-is-finops/)
* **AI Prompt Bank:** Keep these handy to share with students who are stuck:
    * *"Write a basic Apache Airflow DAG in Python that runs daily at 2 AM and executes a simple BashOperator."*
    * *"Explain the difference between Kubernetes Resource Requests and Limits, and how setting them incorrectly drives up cloud costs."*