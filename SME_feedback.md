# SME Feedback: Cost Optimisation and Resource Management
**Module:** Week 7, Class 14 — Cost Optimisation and Resource Management  
**Reviewer:** Sumit Rahman  
**Documents Reviewed:** README.md (student-facing), Instructor Guide

---

## Overall Assessment

This is a practical and well-motivated module. The Apache Airflow introduction is the
clearest technical explanation in Unit 4, the DAG visualisation is the most useful
diagram in the course for AI pipeline work, and the `trigger_rule="all_done"` pattern
for GPU termination is exactly the kind of production gotcha that students won't find
in beginner tutorials. The FinOps framing (ROI, ephemeral compute) is correct and
will resonate with the mid-career audience.

The lesson has one significant deprecated API issue and the recurring lab placeholder.
The deprecated `schedule_interval` parameter used in the lesson's DAG example will
raise a deprecation warning in Airflow 2.x and an error in Airflow 3.x — students who
copy the lesson's code verbatim will hit this on their first run.

---

## 1. Structural Issues

**Issue 1.1 — Redundant Mermaid Diagrams**

The Airflow DAG visualisation from Section 2 (`Start -> Provision GPU -> Training ->
Terminate GPU`) is reproduced in the instructor guide's delivery tips section, and a
simpler version of the same graph appears again in Section 3. Three variations of the
same cost-saving lifecycle diagram add length without adding information. Recommend
keeping the richest version (Section 2) and removing the simpler duplicates.

---

## 2. Alignment with Syllabus and Competency Map

| Learning Objective | In README? | In Instructor Guide? | Notes |
|---|---|---|---|
| Identify cost drivers in AI infrastructure management | ✅ Section 1 | ✅ Yes | Idle GPU, egress, over-provisioning are the right three |
| Apply monitoring and automation techniques to reduce spend | ✅ Section 2 | ✅ Yes | Airflow DAG example is concrete and correct |
| Evaluate performance/scalability/cost balance | ✅ Section 3 | ✅ Yes | Trade-off triangle is clear |

All three LOs are addressed. Coverage is proportionate.

---

## 3. Technical Content

**Issue 3.1 — `schedule_interval` is Deprecated in Airflow 2.x (Breaking in 3.x)**

The lesson's DAG example uses:
```python
with DAG('cost_optimised_ai_training', start_date=datetime(2024, 1, 1),
         schedule_interval='@daily') as dag:
```

`schedule_interval` was deprecated in Airflow 2.4 and will raise an error in Airflow 3.x.
The correct parameter is `schedule`:

```python
with DAG('cost_optimised_ai_training', start_date=datetime(2024, 1, 1),
         schedule='@daily') as dag:
```

Students who copy the lesson's code into a current Airflow installation will see a
`RemovedInAirflow3Warning` deprecation notice and in future installations an outright
error. This should be corrected in the lesson material.

**Issue 3.2 — `trigger_rule="all_done"` Not Explained**

The lesson's DAG example includes `trigger_rule='all_done'` on the GPU termination step
with the comment "Runs even if training fails, preventing infinite billing." This is
correct and important — but the lesson does not explain what trigger rules are or why
the default (`all_success`) would cause the termination to be skipped on failure.

One sentence in Section 2 would close this gap: *"By default, a Kubernetes task only
runs if all upstream tasks succeeded. Setting `trigger_rule='all_done'` means the task
runs regardless of upstream success or failure — essential for cleanup tasks like GPU
termination."*

**Issue 3.3 — Spot Instance Guidance Needs Workload-Type Framing**

Section 3 introduces Spot Instances as "cheap but can be interrupted" versus On-Demand
as "expensive but reliable." The lesson does not provide a decision rule for which
workloads are appropriate for Spot. The instructor guide's troubleshooting table gives
the key insight ("perfect for resumable batch training but terrible for real-time
chatbots") — this framing should be in the student-facing Section 3, not just the
instructor troubleshooting table.

**Issue 3.4 — Data Egress Fees Mentioned Without Quantification**

Section 1 lists data egress as a cost driver but does not give any numbers. For AI
workloads, data egress costs can be substantial: AWS charges $0.09/GB out of us-east-1.
A training run that reads 500GB from S3 in the same region is free, but the same run
in eu-west-1 (cross-region) costs $45 in transfer fees alone — dwarfing the compute
cost for a short training job. A one-sentence example would make this concrete.

---

## 4. Instructor Guide

**Issue 4.1 — Airflow Setup Time Not Addressed**

The pacing guide allocates 40 minutes to a "Live demonstration: Show a simple Apache
Airflow DAG." Installing and starting Airflow requires:
- `pip install apache-airflow` (~5 minutes)
- `airflow db migrate` (~1 minute)
- Creating an admin user (~30 seconds)
- Starting webserver + scheduler in separate terminals

This is a 7–10 minute setup that must be done before the demo begins, not during it.
The instructor guide should note this as a prerequisite: *"Airflow must be running
before the session starts. Start webserver and scheduler 10 minutes before the demo."*

**Issue 4.2 — "DAGs not appearing in the UI" Troubleshooting Note is Critical**

The troubleshooting table entry for "DAGs not appearing in the UI" (run `python
dags/my_dag.py` to catch syntax errors; verify the DAGs folder path) is the most
common Airflow beginner issue. This should appear in the delivery tips, not just the
troubleshooting table, since students are almost guaranteed to encounter it during
the 60-minute lab.

---

## 5. Theory-to-Lab Alignment Recommendations

**Recommendation 5.1 — Fix `schedule_interval` to `schedule` in Lesson DAG Example**  
One character change, high priority. Stops the first error students encounter from
being a lesson typo.

**Recommendation 5.2 — Add `trigger_rule` Explanation to Section 2**  
One sentence. Prevents the "why did my GPU not terminate?" incident that several
students will encounter.

**Recommendation 5.3 — Add Spot Instance Decision Rule to Section 3**  
"Spot Instances are appropriate for: batch jobs with checkpointing (model training,
data pipelines), non-customer-facing compute. Not appropriate for: real-time inference
APIs, any workload where a mid-run interruption is unrecoverable."

---

## Summary of Action Items

| Priority | Issue | Type | Owner |
|---|---|---|---|
| High | Fix `schedule_interval='@daily'` to `schedule='@daily'` in DAG example | Fix | Content Author |
| High | Add `trigger_rule` explanation to Section 2 | Fix | Content Author |
| Medium | Add Spot Instance decision rule (batch OK, real-time not) to Section 3 | Fix | Content Author |
| Medium | Note Airflow setup prerequisites in instructor guide pacing | Fix | Content Author |
| Medium | Elevate "DAGs not appearing" troubleshooting to delivery tips | Fix | Content Author |
| Low | Remove redundant DAG lifecycle diagrams (keep richest, remove duplicates) | Fix | Content Author |
| Low | Add data egress cost quantification example to Section 1 | Recommendation | Content Author |
