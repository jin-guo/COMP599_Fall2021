# Group Project: Movie Recommendations

<em>Credit: This project is adapted from CMU 17-445/645 developed by [Christian Kästner](http://www.cs.cmu.edu/~ckaestne/) et. al.</em>

## Overview

In this project, you will implement, evaluate, operate, monitor, and evolve a recommendation service for a scenario of a movie streaming service with about 1 million customers and 27k movies (for comparison, Netflix has about 180 million subscribers and over 300 million estimated users worldwide and about 4000 movies or 13k titles worldwide).

The focus of this assignment is to operate a recommendation service in production, which entails many concerns, including deployment, scaling, reliability, drift and feedback loops.

The assignment has three milestones and a final project presentation.

## Overall mechanics and infrastructure

**Teamwork:** You will work on this project in your assigned teams. Each team is intentionally formed by people with different ML and SE background. As a team, you will use a shared GitHub repository and a virtual machine to coordinate your work. Please establish a way of communication and collaboration that works for your team -- for example, the provided Slack channel and clear notes from online meetings that include agreed assignments are very useful. Use the first milestone to identify strength and weaknesses, fill missing gaps in background knowledge, and teach each other about tools and practices that are effective for this task. We do not expect that all team members contribute equally to each part of the project, but we expect that all team members make every effort to be a good team member (attend meetings, prepared and cooperative, respect for other team members, work on assigned and agreed tasks by agreed deadlines, reaching out to team members when delays are expected, etc). We will regularly check in about teamwork and perform peer grading to assess team membership of individual students (see this [paper](http://www.rochester.edu/provost/assets/PDFs/futurefaculty/Turning%20Student%20Groups%20into%20Effective%20Teams.pdf) page 28-31 for procedure details).

**Milestones:** For each milestone and the final presentation, there are separate deliverables below (the details for Milestones 2 and 3 will be added soon). The milestones are typically mostly checkpoints to ensure that certain functionality has been delivered. 

**Infrastructure:** You receive data about movies and users through an API 2) and can access log files from past movie watching behavior through a Kafka stream. Note that we now provide separate streams for each team (`movielog<TEAMNR>`). You will offer a prediction service on your virtual machine. You may build a distributed system with multiple machines where you use the provided virtual machine only as API broker or load balancer. 

**Provided data:** We provide an event stream (Apache Kafka) of a streaming service site that records server logs, which include information about which user watched which video and ratings about those movies. The log has entries in the following format:

* `<time>,<userid>,recommendation request <server>, status <200 for success>, result: <recommendations>, <responsetime>` – the user considers watching a movie and a list of recommendations is requested
* `<time>,<userid>,GET /data/m/<movieid>/<minute>.mpg` – the user watches a movie; the movie is split into 1-minute long mpg files that are requested by the user as long as they are watching the movie
* `<time>,<userid>,GET /rate/<movieid>=<rating>` – the user rates a movie with 1 to 5 stars

In addition, we provide read-only access to an API to query information about users and movies. Both APIs provide information in JSON format, which should be mostly self-explanatory. Note that movie data, including “vote_average”, “vote_count” and “popularity” fields come from some external database and do not reflect data on the movie recommendation service itself. In addition, you may collect data from external data sources not provided by the course staff; for example, ids for [IMDB](https://www.imdb.com/) and [TMDB](https://www.themoviedb.org/) are provided for each movie.

You do not need to use all provided data, but should use most of it and not down-sample too much for the final model. Plan for the fact that data gathering and cleaning data may take some time; the provided raw data is fairly large and you may need some time to download and process it.

**Languages, tools, and frameworks:** Your team is free to chose any technology stack for any part of this assignment. However, it should run within a Docker container. That way, you have the freedom to install and deploy any packages and dependencies you might want. You also may use external data and services (e.g. cloud services) as long as you can make them also available to the course staff. Whenever you set up tools or services, pay some attention to configuring them with reasonable security measures.

**Documentation and reports:** For all milestones, you must document your design decision and implementation. It is also a good idea to write general documentation that is useful for the team, in a place that is shared and accessible to the team (e.g., README.md or wiki pages on GitHub). Conversely it may be a good idea to include text or figures you write for reports as part of the project documentation. Feel free to link to more detailed documentation from your report or simply copy material from existing documentation into the report.

## Milestone 1: First Model Deployment

**Learning goals:**

* Collect data from multiple sources and engineer features for learning
* Apply state of the art machine learning tools
* Deploy a model inference service 
* Practice teamwork and reflect on the process

**Tasks:** Get familiar working as a team. Take notes at team meetings about how you divide the work.

Train a model that can make movie recommendations for a specific user. There are many different forms of offering such services that may take past ratings or watch behavior and the similarity between users or between movies into account. You might find the following recourses useful (depending on your familiarity of this topic), including [online tutorials](https://developers.google.com/machine-learning/recommendation), [blog posts](https://towardsdatascience.com/recommender-systems-in-practice-cef9033bb23a), and [articles](https://www.wired.co.uk/article/how-do-netflixs-algorithms-work-machine-learning-helps-to-predict-what-viewers-will-like). 

Build a model inference service that provides movie recommendations on request given your learned model. Specifically, we will start to send http calls to `http://<ip-of-your-virtual-machine>:8082/recommend/<userid>` to which your service should respond with an ordered comma separated list of *up to 20 movie IDs* in a single line; we consider the first movie ID as the highest recommended movie. *You can recognize whether your answer has been correctly received and parsed by a corresponding log entry in the Kafka stream.* 

Note that users of the streaming service will immediately see recommendations you make and your recommendations may influence their behavior.

For this milestone, we do not care about specifics of how you learn or deploy your model, how accurate your predictions are, or how stable your service is.

**Deliverables:** Submit your code to Github and tag it with `M1_done` and submit a short report by email to Jin and Deeksha (email addresses can be found on this repository's root page) containing the following information:

* Learning (1 page max): Briefly describe what kind of machine learning technique you use and why. Provide a pointer to your implementation where you train the model (e.g. to GitHub or other services). 
* Inference service (1 page max): Briefly describe how you implemented the recommendation service and how you derive a ranking from your model. Provide a pointer to your implementation (e.g. to GitHub or other services).
* Team process and meeting notes (1 page max): Briefly describe how your team organizes itself. What communication channels do you use? How do you divide the work? How do you assign responsibilities? Provide pointers to notes taken at team meetings; the notes should describe how work was divided. 

**Grading:** This milestone is worth 100 points: 10 for the description of the learning process and 20 for providing an implementation, 10 for the description of the inference service and 20 for an implementation, 20 points for actually answering the to the API queries, and 20 points for providing the quested documentation about teamwork. For the most part, we will only assess whether the work was completed, not attempt to evaluate the quality of the work.

## Milestone 2: Model and Infrastructure Quality

**Learning goals:**

* Test all components of the learning infrastructure
* Build infrastructure to assess model and data quality
* Build an infrastructure to evaluate a model in production
* Use continuous integration to test infrastructure and models

**Tasks:** First, evaluate the quality of your model both offline and online. For the offline evaluation, consider an appropriate strategy (e.g., suitable accuracy measure, training-validation split, important subpopulations, considering data dependence). For the online evaluation, design and implement a strategy to evaluate model quality in production (e.g., chose and justify metric, collect telemetry).

Second, if you have not done so already, migrate your learning infrastructure to a format that is easy to maintain and test. That will likely involve moving out of a notebook and splitting code for different steps in the pipeline into separate modules that can be called independently. Test all steps of your pipeline so that you have reasonable confidence in the correctness, robustness, and possibly other qualities of your learning solution. Also test the correctness of your inference service. Design, implement, and test a strategy to deal with data quality problems.

Finally, automate all your tests in a continuous integration platform. The platform should automatically build and test your pipeline and inference service implementation and create test and coverage reports. The platform should also automatically trigger learning and evaluating a new model whenever pipeline code changes and report offline accuracy results.

**Technical details:** At this point, we do require that you work regularly with Git for all your code and changes. Make reasonably cohesive commits with appropriate commit messages. We recommend adopting a process with your team in which you use pull requests to review and integrate changes, and leverage Issues and begin discussion threads for communication about design and the code.

The pipeline refers to all parts of the learning process. For example, it should have functionality to gather training/evaluation data from the Kafka stream and the provided APIs (and possibly other data sources), to clean data, and extract features, train models, evaluate predictions, serialize models, serve models, and collect telemetry. All those steps should be reasonably robust and repeatable. You may store intermediate results in files or databases on your virtual machine if you like, but you should be able to recreate that data from scratch with your infrastructure. Overall, the implementation should make it easy to run the pipeline for experimentation (e.g., after changing model parameters) or on more recent data.

Evaluate *infrastructure code quality* by testing all steps in your model learning and evaluation pipeline. Use automated unit tests and report test coverage. For unit testing consider Python's builtin `unittest` or the external `nose2` or `pytest`; `coverage.py` or `Pytest-cov` can be used to measure coverage. You may want to refactor your code, e.g., extract the code that parses a line from Kafka as a separate function, to make it
easier to test. You may consider the use of mocks or stubs for testing. Make decisions about how much testing is appropriate to gain confidence in the correctness and robustness of your solution.

Set up an continuous integration service. You could install Jenkins on your virtual machine or use a cloud service, such as Travis-CI, Circle-CI or others. You may use the same or different services for testing the infrastructure and automating the learning process.

You do not need to create a visual frontend for this milestone, but model-quality reports and continuous-integration reports should be stored in a readable format (preferably machine readable, e.g., JSON, for future automation).

For data quality, focus on data schema issues, missing data, and duplicate data; you do not need to attempt to detect drift. During and after this assignment, we may inject data quality issues in the Kafka stream or provided API or we may make invalid requests to your API. Try to make your service robust, such that it continues to work despite such problems.

If you face storage issues, **please refer to this [post](https://github.com/COMP599Fall2021/IssuesAndQuestions/issues/10)**. If you still face issues or hit other resource limits of your virtual machine, please create a new issue in the repository as soon as possible.

**Deliverables:** Submit your code to Github and tag it with `M2_done` and submit (by email to Jin and Deeksha) a short report that describes the following:

* Offline evaluation (1 page max): Briefly explain how you conduct your offline evaluation. This just include a description of the validation/test data and how it was derived (including a discussion of data dependence and important subpopulation if appropriate) and a brief description and justification of the used metric. Include or link to evaluation results in your report. Provide a pointer to the corresponding implementation in your code (preferably a direct GitHub link).
* Data quality (0.5 pages max): Briefly describe the steps you have taken with regard to data quality and provide a pointer to the corresponding implementation in your code (preferably a direct GitHub link).
* Pipeline implementation and testing (1 page max): Briefly describe how you structured the implementation of your pipeline and how you conducted testing. Include or link to a coverage report. Briefly argue why you think the testing is adequate. Provide a pointer to the corresponding implementation and test suite in your code (preferably a direct GitHub link).
* Continuous integration (0.5 pages max): Describe your continuous integration setup both for infrastructure testing and for automatically training and evaluating models. Provide a pointer to the service (and credentials if needed to access the platform).

**Grading:** This milestone is worth 100 points: 20 for the pipeline implementation and corresponding code and commit quality, 20 points for data quality checks, 20 points for suitable offline model quality assessment, 20 points for good infrastructure testing and an adequacy argument, and 20 points for using continuous integration.

## Milestone 3: Monitoring and Continuous Deployment

**Learning goals:**

* Deploy a model inference service with containers
* Build and operate a monitoring infrastructure for system health and model quality
* Build a continuous delivery pipeline with canary releases and deploy model updates without downtime
* Infrastructure for automatic periodic retraining of models
* Version and track provenance of training data and models

**Tasks:** After gaining confidence in your infrastructure quality and automating essential tasks, we will now focus on deployment, versioning, and monitoring. 

First, containerize your model inference service (and possibly other parts of your infrastructure) and add support for switching between different versions without downtime (e.g., adding a load balancer).

Second, set up a monitoring infrastructure that monitors the health of your recommendation service (including availability) and the quality of its predictions. You might want to set up automated alerts if problems are detected.

Third, build a continuous deployment pipeline where models are automatically pushed into production via a [canary release](https://landing.google.com/sre/workbook/chapters/canarying-releases/) process once they passed offline quality checks. Automate the entire process so that the model learning and deployment process can be triggered with a single command-line call. Further automate the process that models are updated and deployed every 3 days with new production data. Test that poor releases are actually aborted without impacting too many users.

Finally, track provenance of your models such that for every prediction your recommendation service makes you can answer: (1) which version of the model has made the prediction, (2) which version of the pipeline code and ML frameworks has been used to train that model, and (3) what data has been used for training that model.

Keep your recommendation service running as much as possible for the remainder of the project. Prefer poor recommendations over missing or very slow answers from your service. We will look at the logs (public, in the Kafka stream) to assess downtime. 

**Technical details:** We recommend Docker to containerize your model serving infrastructure. You can package the model inside the container or have the container load the model from an external resource (e.g., mounted file system or web server). Your continuous integration/deployment infrastructure should automatically build a new container when needed. You can write your own simple load balancer in 10 lines of Python or Node.js code with `flask` or `express`, if needed, so you can switch between multiple models without downtime.

We recommend to mostly use existing infrastructure for monitoring, such as `Grafana`. You may also use external cloud services if you prefer. Monitor at least availability of your service (e.g., by analyzing the Kafka logs) and the model quality through online evaluation metrics.

For canary releases, you can write your own infrastructure or use an external library or service (e.g., LaunchDarkly or split.io). Releases may be triggered by code changes from your continuous integration service, manually from the command line, or periodically from a scheduler (e.g., `cron`). You could use your own simple load balancer to route traffic to different model servers depending on which users should see which model. You can use your existing telemetry data to make decisions about continuing or aborting the release. Automatically send emails for successful or aborted releases.

Regarding provenance, you have full flexibility. You may use a tool like [`dvc`](https://dvc.org/) or write your own mechanisms. The key point is to being able to track all inputs used for a prediction when needed, e.g., for debugging. Consider tradeoffs among different qualities, such as the amount of data stored and the effort in retrieving it.

Although we do not set explicit requirements for quality assurance, we suggest that you continue to write test cases for your infrastructure code and conduct code reviews within your team.

If you hit resource limits of your virtual machine, contact the course staff.

**Deliverables:** Submit your code to Github and tag it with `M3_done` and submit (by email to Jin and Deeksha) a short report that describes the following:

* Containerization (0.5 pages max): Briefly describe how you containerized and deployed your inference service and where/how you automatically create containers as part of the continuous integration process.  Provide a pointer to the Dockerfile(s) and other relevant implementations (preferably a direct GitHub link).
* Monitoring (0.5 pages text max): Briefly describe how you set up your monitoring infrastructure and what you monitor and whether and why you set alerts. Include a screenshot of your dashboard showing at least availability and model quality measures. Provide pointers to the corresponding code/infrastructure (preferably a direct GitHub link) and explain how we can access your dashboard (include credentials if needed).
* Online evaluation (1 page max): Briefly describe the metric used for evaluating model quality in production, the telemetry data collected, and the operationalization of the metric. Include or link to evaluation results in your report. Provide a pointer to the corresponding implementation in your code (preferably a direct GitHub link).
* Canary releases (1 pages max): Briefly describe how you automate deployments and perform canary releases. Describe how you trigger releases and how the canary release process is implemented (e.g., load balancer, metrics, when to abort). If statistics are used to make decisions in the process explain and justify their use. Include evidence of at least one successful release and one aborted release. Provide pointers to your implementation/infrastructure and log files produced.
* Provenance (2 pages max): Describe how you version and track provenance of models. Explain how you can, for any past recommendation, identify the model version, the used pipeline version, and the used training data. Give a concrete example with one past recommendation. Provide sufficient pointers such that the course staff could also identify the corresponding information for a given recommendation.
* Team contributions (2 pages max): Briefly describe how your team organized itself. How did you divide the work? What responsibilities did each person have, and what contributions did each person make? Provide pointers to notes taken at team meetings, as well as evidence of significant contributions, such as via commits on Github.

**Grading:**
Grading is divided into two parts:

1. Milestone Score (max 100 points): 5 for containerization, 30 for monitoring availability and model quality (15 for service avaialablity and 15 for model quality), 40 for automated canary releases (including 10 for automatic model builds, 15 for incremental rollouts with automated decisions, 5 for automated notifications of successful and failed releases, 5 for evidence a successful and a correctly aborted release, and 5 for automatic regular releases), 25 points for provenance tracking.

2. Contribution Score (integer score between 6-10 based on the Team Contributions and grader's observations of existing evidence):
    - 6: Token contributions only
    - 7: Some participation, though limited in initiative and ideas
    - 8: Few active contributions
    - 9: Regular active contributions
    - 10: Regular, active contributions, with participation (even in terms of discussion) in most aspects of the project

Each team member will receive a final grade for this Milestone 3 calculated as: `(contribution/10)*milestone`

## Final Report and Presentation

**Learning goals:**

* Analyze the feedback loops, fairness, and adversarial attacks in machine-learning systems
* Design and implement a monitoring strategy to detect feedback loops and attacks
* Reflect on the quality and operation of an AI-enabled system

**Tasks:** Up to this point, the project focused on implementing and testing functionality, but you also now have multiple weeks of making recommendations and observing their effects. This final part has three steps: (1) reflection on the infrastructure and teamwork, (2) analyzing feedback loops, fairness, and potential attack points, and (3) a presentation.

First, reflect on the entire group project and discuss which decisions were hard and why, which decisions you would change in retrospect, and what you would do differently if you were building this for an actual company (probably with more time but also with higher stakes). Also reflect on your experience of working as a team (remotely or in-person).

Second, consider feedback loops due to recommendations (positive or negative), fairness, and the potential for attacking the recommendation system (e.g., poisoning and evasion attacks). Think about what responsibility you might have as company and as authors of the recommendation system. Plan how you would detect a feedback loop and what you can do to avoid negative consequences once detected. Analyze the system logs or models for traces of either feedback loops, fairness issues, or attacks.

Third, create a 15-20 min presentation to the class presenting the project and your experiences. The presentation should cover some key design decisions and some results (e.g., quality measures or availability) and reflection, but generally you are free to chose how you focus your presentation. Note that all teams worked on the same project, so you can assume familiarity with the task.

**Technical details:** Good reflections are grounded in concrete experience and the specifics of the project. They avoid mere superficial statements and truisms. We are looking for honest reflections that are open about potential issues.

For analyzing feedback loops, fairness, and attacks, better solutions will follow a more systematic process than just brainstorming: It is useful to go back to requirements and analyze assumptions and perform some risk analysis. We suggest you analyze past recommendations and user behavior, for example, changes in user behavior over time, recommendation quality differences for different populations, drift in recommendation requests or user behavior. We have no requirements for how to conduct this analysis but recommend to explore the data and share the results. We encourage you to look for feedback loops or attacks that are not actually occurring or detect issues that we are not simulating.

For the presentation, we recommend that you prepare slides and practice timing. All team members should have an active part in the presentation. You may prerecord the presentation and play the recording for the final presentation if you prefer.

**Deliverables:** Present your work in the dedicated session and submit a final report (by email to Jin and Deeksha) that describes the following:

* **Conceptual Analysis of Potential Problems** (3 pages max): Describe the process you used to analyze possible feedback loops, fairness, and attacks. Identify at least 2 potential issues each. For each feedback loop, explain what might happen and what the positive or negative consequences are, and how it could be detected. For each fairness issue, describe the potential problem, the used notion of fairness, how it could be detected, and how it can be reduced. For each potential attack, describe the attack scenario, how it could be detected, how it could be mitigated or made harder to exploit, and what could be done once detected.
* **Analysis of Problems in Log Data** (2 pages max): Briefly describe how you analyzed one potential issue (either feedback loop, fairness, or attack) in the telemetry data of your system. Summarize your key findings, including negative results. Provide pointers to the artifacts behind your analysis for details (ideally links to files on Github).
* **Reflection on Recommendation Service** (1 page max): Looking back at the entire project in which you have designed, implemented, deployed, and monitored the recommendation service. What parts were the most challenging? Which aspects are still unstable and would require additional investment if you had to deploy the recommendation service at scale in production? How would you address these issues if you had more time and more resources?
* **Reflection on teamwork** (1 page max): Think back on your team's teamwork throughout this project. What went well or less well in the team assignments? What were some of the main challenges you faced in teamwork?  What could have been done better in your future collaboration in other teams? 
* **Team contributions and meeting notes** (2 pages max): Briefly describe how do you divided the work? What responsibilities did each person have, and what contributions did each person make? Provide evidence of significant contributions, such as via commits on Github. Provide pointers to notes taken at team meetings; the notes should contain who contributed to the discussions, the flow of the discussions, how ideas were generated, the responsibilities that each team member handled, etc.

**Grading:**
Grading is divided into two parts:

1. The presentation is worth 20 points, the final report 30 points, 30 for the conceptual problem analysis and discussion of detection and mitigation strategies, 20 for analyzing the problems with telemetry data.

2. Contribution Score (integer score between 6-10 based on the Team Contributions and grader's observations of existing evidence):
    - 6: Token contributions only
    - 7: Some participation, though limited in initiative and ideas
    - 8: Few active contributions
    - 9: Regular active contributions
    - 10: Regular, active contributions, with participation (even in terms of discussion) in most aspects of the project

Each team member will receive a final grade for this Final Report and Presentation calculated as: `(contribution/10)*milestone`

