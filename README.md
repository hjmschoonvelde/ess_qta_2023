# Essex Summer School 2023 — Quantitative Text Analysis

This page contains the materials for the Essex Summer School 2023 course *Introduction to Quantitative Text Analysis*. Materials will be added as we go along.

Instructor: [Martijn Schoonvelde](http://mschoonvelde.com); TA: [Abdullah Arslan](mailto:abdullah.arslan10068@gmail.com)

You can find the syllabus [here](Syllabus_QTA.pdf).

## Communication

To facilitate communication and interaction throughout the course we will make use of a dedicated [Slack channel](https://essqta23.slack.com).

## Slides

| Date        | Slides           |  Date        | Slides           |
| ------------- |:-------------:| ------------- |:-------------:|
| July  10   | [Link]( Slides/Slides_QTA_1.pdf )| July  17   | <!--[Link]( Slides/Slides_QTA_6.pdf) --> |
| July  11   | [Link](Slides/Slides_QTA_2.pdf )| July  18   | <!--[Link](Slides/Slides_QTA_7.pdf) --> |
| July  12   | <!--[Link](Slides/Slides_QTA_3.pdf ) --> | July  19   | <!--[Link](Slides/Slides_QTA_8.pdf) --> |
| July  13   | <!--[Link](Slides/Slides_QTA_4.pdf)  -->| July  20   |<!--[Link](Slides/Slides_QTA_9.pdf) --> |
| July  14   | <!--[Link](Slides/Slides_QTA_5.pdf) -->| July  21   | <!--[Link](Slides/Slides_QTA_10.pdf)  --> |


## Lab Sessions

| Date        | Link           | Solutions           |    
| ------------- |:-------------:|:-------------:|
| July  10   | [.md]( Lab_sessions/Day_1/Lab_Session_QTA_1.md ) [.Rmd]( Lab_sessions/Day_1/Lab_Session_QTA_1.Rmd ) | [.md](Lab_sessions/Day_1/Lab_Session_QTA_1_Answers.md) [.Rmd](Lab_sessions/Day_1/Lab_Session_QTA_1_Answers.Rmd) |
| July  11   | [.md](Lab_sessions/Day_2/Lab_Session_QTA_2.md ) [.Rmd](Lab_sessions/Day_2/Lab_Session_QTA_2.Rmd ) | [.md](Lab_sessions/Day_2/Lab_Session_QTA_2_Answers.md) [.Rmd](Lab_sessions/Day_2/Lab_Session_QTA_2_Answers.Rmd) |
| July  12   | <!--[Script](Lab_sessions/Day_3/Lab_Session_QTA_3.md ) -->| <!--[Exercise solution](Lab_sessions/Day_3/Lab_Session_QTA_3_Answers.md) --> |
| July  13   | <!--[Script](Lab_sessions/Day_4/Lab_Session_QTA_4.md ) -->|<!-- [Exercise solution](Lab_sessions/Day_4/Lab_Session_QTA_4_Answers.md) --> |
| July  14   | <!--[Script](Lab_sessions/Day_5/Lab_Session_QTA_5.md) -->| <!-- [Exercise solution](Lab_sessions/Day_5/Lab_Session_QTA_5_Answers.md) --> |
| July  17   | <!--[Script](Lab_sessions/Day_6/Lab_Session_QTA_6.md) -->| <!--[Exercise solution](Lab_sessions/Day_6/Lab_Session_QTA_6_Answers.md) --> |
| July  18   | <!--[Script](Lab_sessions/Day_7/Lab_Session_QTA_7.md) -->| <!--[Exercise solution](Lab_sessions/Day_7/Lab_Session_QTA_7_Answers.md) --> |
| July  19   | <!--[Script](Lab_sessions/Day_8/Lab_Session_QTA_8.md) -->| <!--[Exercise solution](Lab_sessions/Day_8/Lab_Session_QTA_8_Answers.md) --> |
| July  20   | <!--[Script](Lab_sessions/Day_9/Lab_Session_QTA_9.md) -->| <!--[Exercise solution](Lab_sessions/Day_9/Lab_Session_QTA_9_Answers.md) --> |
| July  21   | <!--[Script](Lab_sessions/Day_10/Lab_Session_QTA_10.md) -->| <!--[Exercise solution](Lab_sessions/Day_10/Lab_Session_QTA_10_Answers.md) --> |

<!-- ## Flash talks

| Name        | Link           | 
| ------------- |:-------------:| 
 -->

## Acknowledgements

For some code and ideas in the lab scripts, I made use of materials by Jos Elkink [here](http://www.joselkink.net/files/POL30430_Spring_2017_lab11.html), and [here](http://www.joselkink.net/files/POL30430_Spring_2017_lab12.html); Wouter van Atteveldt and Kasper Welbers [here](https://github.com/ccs-amsterdam/r-course-material/blob/master/tutorials/r_text_nlp.md) and [here](https://github.com/ccs-amsterdam/r-course-material); and **quanteda** [tutorials](https://tutorials.quanteda.io/) here. I thank Stefan Müller for sharing his lab session materials for his QTA course at UCD. Thanks to all!

## Course schedule


*Day 1 - July 10*

 - **Lecture**: What is quantitative text analysis? What will you learn in this course? Developing a corpus.
 
-  **Lab**: Working in RStudio Cloud. Working with libraries in R. Working with RMarkdown. 

- **Readings**
  - Benoit (2020). Text as Data: An Overview. Handbook of Research Methods in Political Science and International Relations. Ed. by L. Curini and R. Franzese. Thousand Oaks: Sage: pp. 461–497.
  - Grimmer, J., & Stewart, B. M. (2013). Text as data: The promise and pitfalls of automatic content analysis methods for political texts. Political Analysis, 21(3), pp. 267–297.

*Day 2 - July 11*

-	**Lecture**: Core assumptions in quantitative text analysis. Representations of text. Preprocessing and feature selection.

-	**Lab**: Working with strings variables. Regular expressions. Cleaning a string vector. Creating a document-feature matrix. 

- **Readings**:
  -  Welbers, K., Van Atteveldt, W., & Benoit, K. (2017). Text analysis in R. Communication Methods and Measures, 11(4): pp. 245–265.
  - Baden, C., Pipal, C., Schoonvelde, M. & van der Velden, M.A.G., (2022). Three Gaps in Computational Text Analysis Methods for Social Sciences: A Research Agenda. Communication Methods and Measures, 16(1): pp. 1–18.

*Day 3 - July 12*

-	**Lecture**: Advanced text representations. Risks of feature selection with unsupervised models. 

-	**Lab**: Importing textual data into R. Introduction to **quanteda** (Benoit _et al._, 2018). Inspecting and visualizing a corpus. 

- **Readings**:
  -  Denny, M.J. & Spirling, A., (2018). Text preprocessing for unsupervised learning: Why it matters, when it misleads, and what to do about it. Political Analysis, 26(2): pp.168–189.
  - Benoit, K., Watanabe, K., Wang, H, Nulty, P., Obeng, A., Müller, & Matsuo, A. (2018).
Quanteda: An R package for the quantitative analysis of textual data. Journal of Open Source Software, 3(30), 774.

*Day 4 - July 13*

-	**Lecture**: Comparing documents in a corpus. Generating insights by combining linguistic features with social science theories. 

-	**Lab**: Examining similarity and complexity of documents. 

- **Readings**
  -  Peterson, A. & Spirling, A., (2018). Classification accuracy as a substantive quantity of interest: Measuring polarization in Westminster systems. Political Analysis, 26(1): pp. 120– 128.
  - Hager, A. and Hilbig, H., (2020). Does public opinion affect political speech? American Journal of Political Science, 64(4): pp. 921--937.

*Day 5 - July 14*

-	**Lecture**: What can we do with dictionaries and how can we validate them? Sensitivity and specificity. 

-	**Lab**: Categorizing texts using off-the-shelf and home-made dictionaries. 

- **Readings**:
  -  Rauh, C., (2018). Validating a sentiment dictionary for German political language—a workbench note. Journal of Information Technology & Politics, 15(4): pp. 319-343.
  - S.-O. Proksch, W. Lowe, J. Wäckerle, and S. N. Soroka (2019). Multilingual Sentiment Analysis: A New Approach to Measuring Conflict in Legislative Speeches. Legislative Studies Quarterly 44(1): pp. 97–131.

*Day 6 - July 17*

-	**Lecture**: Human coding and document classification using supervised machine learning. Evaluating a classifier. 

-	**Lab**: Binary classification of documents using a Naïve Bayes classifier.

- **Readings**:
  -  Daniel Jurafsky and James H. Martin (2020). Speech and Language Processing: An Introduction to Natural Language Processing, Computational Linguistics, and Speech Recognition. 3rd edition: Chapter 4.
  - Müller, S., (2020). “Media coverage of campaign promises throughout the electoral cycle.” Political Communication: pp. 1–23.

*Day 7 - July 18*

-	**Lecture**: Supervised, semi-supervised and unsupervised approaches to place text on an underlying dimension. 

-	**Lab**: Wordfish, Wordscores and Latent Semantic scaling.

- **Readings**:

  - Slapin J. & Proksch S. (2008). A scaling model for estimating time-serial positions from texts. American Journal of Political Science 52: pp. 705–722.
  - Watanabe, K., (2021). Latent semantic scaling: A semisupervised text analysis technique for new domains and languages. Communication Methods and Measures, 15(2), pp.81-102.
  - Schwemmer, C. and Wieczorek, O., (2020). The methodological divide of sociology: Evidence from two decades of journal publications. Sociology, 54(1): pp.3-21.

*Day 8 - July 19*

-	**Lecture**: Understanding topic models. Discussing their pros and cons. 

-	**Lab**: Latent Dirichlet Allocation (LDA) and Structural topics models (STM).

- **Readings**:
  - Blei, D. M. (2012). Probabilistic topic models. Communications of the ACM, 55(4), pp. 77–84.
  - Roberts, M et al. (2014). Structural topic models for open-ended survey responses. American Journal of Political Science, 58(4), pp. 1064–1082.

*Day 9 - July 20*

-	**Lecture**: New developments in data.  Multilingualism. Automated speech recognition. Images as data.

-	**Lab**: Linguistic preprocessing of text. POS tagging and lemmatizing using **udipe** (Wijffels, 2022)

- **Readings**:
  - Proksch, S.O., Wratil, C. and Wäckerle, J., (2019). Testing the validity of automatic speech recognition for political text analysis. Political Analysis, pp. 1–21
  - De Vries, E., Schoonvelde, M. & Schumacher, G., (2018). No longer lost in translation: Evidence that Google Translate works for comparative bag-of-words text applications. Political Analysis, 26(4), pp. 417–430.
  - Schwemmer, C., Unger, S. and Heiberger, R., (2023). 15. Automated image analysis for studying online behaviour. Research Handbook on Digital Sociology, p.278.

*Day 10 - July 21*

-	**Lecture**: Word embeddings. Transformer-based text classification. Concluding remarks. 

-	**Lab**: Training a word embeddings model and inspecting document vectors using **text2vec** (Selivanov _et al_ 2022)

- **Readings**:
  - Rodriguez, P.L. and Spirling, A., (2022). Word embeddings: What works, what doesn’t, and how to tell the difference for applied research. The Journal of Politics, 84(1), pp.101-115.
  - Rodman, E., (2020). A Timely Intervention: Tracking the Changing Meanings of Political Concepts with Word Vectors. Political Analysis, 28(1): pp. 87–111.
  - Chan, C.H., (2023). grafzahl: fine-tuning Transformers for text data from within R. Computational Communication Research, 5(1), p.76-84.


