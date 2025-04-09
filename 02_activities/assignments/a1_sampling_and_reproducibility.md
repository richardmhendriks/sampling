# ASSIGNMENT: Sampling and Reproducibility in Python

Read the blog post [Contact tracing can give a biased sample of COVID-19 cases](https://andrewwhitby.com/2020/11/24/contact-tracing-biased/) by Andrew Whitby to understand the context and motivation behind the simulation model we will be examining.

Examine the code in `whitby_covid_tracing.py`. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times

# Author: RICHARD HENDRIKS

```
Please write your explanation here...

```
1. Identify all stages at which sampling is occurring in the model. Describe in words the sampling procedure, referencing the functions used, sample size, sampling frame, any underlying distributions involved, and how these relate to the procedure outlined in the blog post.

The following locations in the model reflect where sampling is occurring:

A. # Infect a random subset of people
infected_indices = np.random.choice(ppl.index, size=int(len(ppl) * ATTACK_RATE), replace=False)

Procedure: simple random sample
Population: 200 (weddings) + 800 (gatherings) = 1000
Sample size: Attack Rate * 1000 = 100
Observational units: persons who could contract COVID-19 (presumed in the original blog post to be all persons with equal likelihood of contracting infection. i.e. none are vaccinated or had a recent case of COVID-19 that would lower their infection rate)
Sampling units: persons who attended either of two outbreak settings, namely a wedding or a gathering (the model implicitly assumes none of the 1000 attended both a wedding and a gathering)
Sampling frame: the lists of persons who attended a wedding or a gathering 
Distributions: see the implicit assumptions noted above in relation to the observational units and sampling units, namely that all are equally likely to contract COVID-19 and that the wedding and gathering populations are distinct, which may not be the case
Relationship to article: this sampling reflects the "cases" in the table in the original blog post


B. # Primary contact tracing: randomly decide which infected people get traced
  ppl.loc[ppl['infected'], 'traced'] = np.random.rand(sum(ppl['infected'])) < TRACE_SUCCESS

Procedure: Simple random sample
Population: 100 cases of persons infected with COVID-19
Sample size: up to 20% of the 100 cases recieving primary contact tracing successfully traced to the source of the COVID-19 infection (I say "up to" because the formula is < 20% and not = 20%) 
Observational units: persons who contracted COVID-19 at a wedding or gathering (again, the model implicitly assumes none of the 100 attended both a wedding and a gathering)
Sampling units: persons infected with COVID-19 who were contacted, available, responsive and capable of responding to primary contact tracing
Sampling frame: persons who tested positive for or were otherwise confirmed to have contracted COVID-19
Distributions: A key assumption here is that the distribution of the sample of persons successfully contact traced has a similar distribution to those persons not successfully contact traced; this is unlikely to be the case, as the latter group may include more/fewer persons unable to respond, unable to recall their recent activities, unwilling to respond, etc. 
Relationship to article: this sampling reflects the imperfection of contact tracing in the article, which is assumed to be 20% successful; the code reflets a slight difference from the article, which indicates 20%

C. # Secondary contact tracing based on event attendance
  event_trace_counts = ppl[ppl['traced'] == True]['event'].value_counts()
  events_traced = event_trace_counts[event_trace_counts >= SECONDARY_TRACE_THRESHOLD].index
  ppl.loc[ppl['event'].isin(events_traced) & ppl['infected'], 'traced'] = True

Procedure: Systematic sampling (i.e. secondary contract tracing follows from primary contact tracing, which is a random starting point, and then secondary systematic sampling is based on a fixed criterion)
Population: persons determined to be infected with COVID-19 who were tested since they attended a wedding or gathering where 2 or more primary contacts successfully traced back to that wedding or gathering
Sample size: indeterminate, since it depends on the frequency of weddings or gatherings where 2 or more successful primary contacts traced back to that wedding or gathering
Observational units: persons who may have contracted COVID-19 at a wedding or gathering
Sampling units: persons (i.e. clusters) who attended a wedding or gathering that at least two other persons were confirmed to have contracted COVID-19 as a result of primary contact tracing
Sampling frame: the lists of persons who attended a wedding or gathering that at least two other persons were confirmed to have contracted COVID-19 as a result of primary contact tracing 
Distributions: There is a potential error in the python code that misrepresents the original analysis. The original code on github is in R and appears to have been translated incorrectly into python. See below for a further explanation.
Relationship to article: this systematic sampling reflects the "secondary contact tracing" that the article assumes is being implemented by health agencies

ORIGINAL R CODE: The original code appears to set the threshold for secondary contact tracing at 2 or more successful primary contacts traced back to a INDIVIDUAL wedding (of size 100) or gathering (of size 10). Because of this difference in event size, it is far more likely that secondary contact tracing will occur in relation to weddings as opposed to gatherings. This is the case since 2 persons at a 100-person wedding is only 2% of the attendees whereas 2 persons at a 10-person gathering is 20% of the attendees. The threshold for triggering secondary contact tracing at a wedding is therefore much lower than for triggering contact tracing at a gathering. It would be prefable to set the secondary trace threshold as a proportion (i.e. a percentage) of the persons attending the event rather than as a raw number of infected persons.

PYTHON CODE: In order to see the effect of this critical assumption the original code, requires the revised code reflect not only the number of weddings and gatherings but also the size of the weddings and gatherings. The python code does not do that. As a result, it appears that the secondary trace threshold for weddings is 2 of 200 and of gatherings is 2 of 800, rather than 2 of 200 and 2 of 10, respectively. 


2. Run the Python script file called whitby_covid_tracing.py as is and compare the results to the graphs in the original blog post. Does this code appear to reproduce the graphs from the original blog post?

First, I presume that the darker red in the graph in the original blog post is just the overlap of the two graph colours, where blue is the true proportion and the pale red is the observed proportion. By "observed proportion" I presume that this is the proportion observed in the modelling not in the real world.

To answer the question, no, the code does not reproduce the results shown in the original blog post.  This is could be due to a difference in the number of runs, although one would expect that 50,000 runs would result in a tighter spread for the observed proportion in the blog post than the 1,000 runs in the code. The difference is likely due to the differences in formulation between the original R code and the python code, as discussed above.


3. Modify the number of repetitions in the simulation to 100 (from the original 1000). Run the script multiple times and observe the outputted graphs. Comment on the reproducibility of the results.

The code does not produce the same results over multiple runs as can be seen in the graphs from repeat runs of the model. This was also the case for 1000 runs of the model in question 2, though less obvious since the larger number of runs reduces the variance in the results. In order to reproduce the wide distribution in the graph of the observed proportion in the original blog post 


4. Alter the code so that it is reproducible. Describe the changes you made to the code and how they affected the reproducibility of the script file. The output does not need to match Whitby’s original blogpost/graphs, it just needs to produce the same output when run multiple times.

Prior to the defining the function, which contains randomization, I set a random seed in the code. This ensures the same results are produced on each run of the model.



## Criteria

|Criteria|Complete|Incomplete|
|--------|----|----|
|Altercation of the code|The code changes made, made it reproducible.|The code is still not reproducible.|
|Description of changes|The author explained the reasonings for the changes made well.|The author did not explain the reasonings for the changes made well.|

## Submission Information

🚨 **Please review our [Assignment Submission Guide](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md)** 🚨 for detailed instructions on how to format, branch, and submit your work. Following these guidelines is crucial for your submissions to be evaluated correctly.

### Submission Parameters:
* Submission Due Date: `23:59 - 09/04/2025`
* The branch name for your repo should be: `assignment-1`
* What to submit for this assignment:
    * This markdown file (a1_sampling_and_reproducibility.md) should be populated.
    * The `whitby_covid_tracing.py` should be changed.
* What the pull request link should look like for this assignment: `https://github.com/<your_github_username>/sampling/pull/<pr_id>`
    * Open a private window in your browser. Copy and paste the link to your pull request into the address bar. Make sure you can see your pull request properly. This helps the technical facilitator and learning support staff review your submission easily.

Checklist:
- [ ] Create a branch called `assignment-1`.
- [ ] Ensure that the repository is public.
- [ ] Review [the PR description guidelines](https://github.com/UofT-DSI/onboarding/blob/main/onboarding_documents/submissions.md#guidelines-for-pull-request-descriptions) and adhere to them.
- [ ] Verify that the link is accessible in a private browser window.

If you encounter any difficulties or have questions, please don't hesitate to reach out to our team via the help channel in Slack. Our Technical Facilitators and Learning Support staff are here to help you navigate any challenges.
