# Workshop abstract voting with Qualtrics and Office

The aim of this notebook is to show some code for generating a Qualtrics voting form for a workshop, as we used in [SNUFA 2022](https://snufa.net/2022).

## Step 1: Collect abstracts

We used [Office Forms](https://forms.office.com) but Google forms or any other system would work as well. We collected author names, emails, abstract text and whether or not you wanted to be considered for poster or talk.

From this we get an Excel file that can be converted to a CSV.

## Step 2: Generate Qualtrics survey

### Qualtrics import file format

[Qualtrics](https://www.qualtrics.com) is a survey tool. I used it because Imperial College has a site license and because it has some nice features for importing forms, which I'll use below.

The main step is generating a text file that can be used to import questions using [Qualtric's somewhat unclearly documented "advanced format"](https://www.qualtrics.com/support/survey-platform/survey-module/survey-tools/import-and-export-surveys/). These look a bit like this:

```
[[AdvancedFormat]]

[[Question:DB]]

<div>
    <h3>Presentation title</h3>
    <p style="font-size: 80%;">
        The text of the abstract here.
    </p>
</div>

[[Question:MC:SingleAnswer]]
[[ID:abstract1yesno]]
I would like to see this as a talk.
[[Choices]]
No
Yes

[[Question:TextEntry:Form]]
[[ID:abstract1comments]
Any comments?
[[Choices]]
Insert comments here.
```

Once you have a file in this format, you can [import it into Qualtrics](https://www.qualtrics.com/support/survey-platform/survey-module/survey-tools/import-and-export-surveys/).

### Using Python and Jinja to generate the form

We use Python and the [Jinja templating engine](https://palletsprojects.com/p/jinja/) to do this. In addition to the above, we insert a ``[Block]`` in between each batch of questions. Here's the code.

```Python
import csv
import jinja2

with open('submissions.csv', newline='') as csvfile:
    csvreader = csv.DictReader(csvfile, dialect='excel')
    submissions = list(csvreader)

template = jinja2.Template('''
[[AdvancedFormat]]
    
{% for sub in submissions %}
{% if sub['Would you prefer a poster or talk (if selected)?']=="Talk preferred" %}

[[Block]]

[[Question:DB]]
    
<div>
    <h3>{{ sub['Presentation title'] }}</h3>
    {% for para in sub['Abstract (please keep under 300 words)'].splitlines() %}
    {% if para.strip() %}
    <p style="font-size: 80%;">
        {{ para }}
    </p>
    {% endif %}
    {% endfor %}
</div>

[[Question:MC:SingleAnswer]]
[[ID:abstract{{ sub.ID }}yesno]]
I would like to see this as a talk.
[[Choices]]
No
Yes

[[Question:TextEntry:Form]]
[[ID:abstract{{ sub.ID }}comments]
Any comments?
[[Choices]]
Insert comments here.
{% endif %}
{% endfor %}
''')

survey = template.render(submissions=submissions)

open('survey.txt', 'w', encoding='utf-8').write(survey)
```

### Importing and tweaking the form in Qualtrics

Having imported the generated ``survey.txt`` file into a new Qualtrics survey, I added randomisation. Asking your audience to vote on all the abstracts might be a bit too much, so we can show each voter a random subset using [Qualtrics' Randomizer](https://www.qualtrics.com/support/survey-platform/survey-module/survey-flow/standard-elements/randomizer/). Create a Randomizer in the survey flow, drag all the imported blocks into it, and ask it to show 10 at a time with the "Evenly Present Elements" option.

You might also want to tweak the options for how the form is displayed, add a logo, intro page, etc.

## Step 3: Publicise and analyse data

Send a copy to all your workshop participants and give them a week to respond. Download the votes and analyse. I'll add some analysis code here once I get round to doing that.
