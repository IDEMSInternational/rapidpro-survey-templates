
[Authoring Guide](https://docs.google.com/document/d/1yTxqPQwJ0BUebF14nU-Pveeb9dD1GIzVtv9AwEmtYbg/edit?tab=t.0)

## Survey templates

We define global templates that are used by default to create surveys with the [rapidpro-flow-toolkit](https://github.com/IDEMSInternational/rapidpro-flow-toolkit). They are as follows:

- `template_survey_wrapper`: Flow rendering all the questions.
	- Receives the following context variables that can be used in the template:
		- `questions`: a list of `SurveyQuestionRowModel`
		- `survey_name`: Name of the survey
		- `survey_id`: ID of the survey (generated from name)
	- In the content index, a `survey` row can have `template_arguments`. If present, these are passed to the `template_survey_wrapper` template when creating a survey.
- `template_survey_question_wrapper`: Question functionality that is common to all input types. Invoked by the survey via `start_new_flow`
	- Receives the fields of the `SurveyQuestionRowModel` as its context variables
	- Currently, it is not possible to pass template arguments to this template.
- `template_survey_question_block_{type}`: For each question input type `{type}`, there is a template to read the user data. These are included into the `template_survey_question_wrapper` via `insert_as_block`
	- Because this template is inserted as a block, any context that is available in `template_survey_question_wrapper` (in particular, `question`) is also available in this template.

The user can overwrite these by defining a template of the same name in the content index, thereby using their own custom templates. There is no constraint on what `{type}` can be, therefore the user can also create their own question types.

## IMPLEMENTATION NOTES
- Expiration: Each survey question is included as a subflow in the survey wrapper, which handles the expiration behaviour (i.e. sending a message and exiting the flow without considering the question as completed, meaning the corresponding completion variable is not set to "yes"). However, the 'postprocessing.flow' feature allows subflows to be entered from a question. If the flow includes a 'wait_for_response' node, users can expire while in the flow. To account for this, the result variable 'next_action' is set to 'expired' in the question and interpreted at the wrapper level, resulting in the same behaviour as expiring while answering the survey question itself. However, the completion variable for the question is set to 'yes' before the post-processing flow, meaning users won't receive it again when they next go through the survey (unlike if they had expired while answering the survey question itself).

