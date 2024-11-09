# Turmerik-TakeHome-Assignment
## Sentiment Analysis and Personalized Messaging for Clinical Trial Recruitment

#### Objective:

The objective of this project is to demonstrate your ability to ethically scrape and analyze web data, utilize sentiment analysis, and leverage AI to personalize communication. You will focus on identifying potential participants for a clinical trial by analyzing sentiments expressed on Reddit.

# Setup
1. Have your conda env prepared and run
```bash
conda create -n sentiment python=3.10
conda activate sentiment
```
2. After activation, go the folder turmerik-ml-project and run
```bash
pip install -r requirement.txt
```
3. Prepare the juputer notebook env (it is recommended using VS Code which will handle installing the notebook kernel for you)

4. Export necessary API keys
```bash
export OPENAI_API_KEY=<YOUR API KEY>
```
or you can paste your openai api key in the personalized-recommendation notebook.

5. Run the notebook
    Please create a script app on reddit to obtain relavant information and replace all the credentials in reddit-scrape.ipynb if you want to run web crawler script.
    Make sure you obtain the right credentials from both reddit and openai and put them in the notebok


# Discussion
1. **Data scraping from reddit**
    As recommended in the requirement, I use praw to scrape reddit website. One challenge is what to scrape and where to scrape and I actually have not used reddit before. So I checked reddit website and found that reddit works based on different subreddits, which resemble different community. To search for related posts, I identify several possible subreddits that might contain comments related to clincal trials. Here is the list(This can be further extened as needed):
    ```python
    ['clinicalresearch', 'health', 'depression', 'mentalhealth', 'science','cancer']
    ```
    Under each subreddit, I use 
    ```python
    ["clinical trial", "research study", "new treatment", "volunteer"]
    ```
    as keywords to do further search.
    My script will retrieve lastest 10 post from each search results together with all comments below it and stored them in json format. 

    An example of collected json will be
    ```json
    {"clinicalresearch+clinical trial": {
        "posts": [
            {
                "subreddit": "clinicalresearch",
                "score": 0,
                "title": "Do I need to use real DOB for clinical trial?",
                "selftext": "I want to join clinical trials as a healthy volunteer, but I\u2019m worried about data breaches because I am a woman in a republican state. Is there any reason I can\u2019t just use a birthdate a few months off of my real one so it doesn\u2019t link to my real EPIC chart? \n\nWould this affect payment at all which would be on a prepaid card or a check? \nWhat about not using my real SSN?\n\nI would never change data that would alter the study (age or history), I just want to know if there\u2019s a way to not have this affiliated with my real EPIC chart and still receive payment without issue.\n\nI don\u2019t want a lecture on HIPAA and data security, just answers please. Thank you!",
                "url": "https://www.reddit.com/r/clinicalresearch/comments/1gmpj7s/do_i_need_to_use_real_dob_for_clinical_trial/",
                "comments": [
                    {
                        "user_name": "kaywhyesay",
                        "time": "2024-11-08 13:44:01",
                        "content": "You will always be asked for a drivers license which has your DOB. It sounds like participating in trials is not the move for you.",
                        "comment_id": "lw4fqyx"
                    } ]
            }
        ]
    }
    }
                    
    ```

2. **Sentiment Analysis**

    The main challenge in this part is how to conduct sentiment analysis, given the resource I have. For sure, we can train a classification model from scratch. However, after searching on the websites, there are few good open source dataset for sentiment analysis presented on the Internet. Therefore, using a pretained model become a good option and I've identified "finiteautomata/bertweet-base-sentiment-analysis" on huggingface which is a bert-based model fine-tuned on sentiment analysis task. The model is not large and is suitable for running locally and is quite aligned with the intended usage in the project.

    Given a text, the model will output **NEG, NEU, POS** together with their probablity. I run the model on all comments and stored them in a pandas dataframe. One thing to notice is that some comments might be too long and exceed the input limit. In that case, I utilized a pretrained language model("sshleifer/distilbart-cnn-6-6" for summarization) to summarize the input, after which it will be fed into the bertweet.

    In general, people have neutral and negative attitudes towards clinical trials. I further divided people into different levels of interests based on the assigned catagory and the probablity score, which might not be ideal since the I manually set the threholds for different levels.

3. **Personalized Recommendation**
    In this part, I mainly adopt gpt-4o-mini to do the task. Specifically I try to use two strategies. One is level-based, which I tell gpt the information from the user (level of interest, original of the comment). The prompting style is shown in the notebook. I sampled 30 case from the dataset and test on it. The example output looks friendly, careful and professional. Another approach I use is precise recommendation, where I want to let gpt determine the level of interest from the users. However, this fails because 4o along with the example I provide might not be enough to determine the level of intention in terms of clinical trial.

4. **Ethical Concerns and Reflection**
    1. Data Scraping
        Since Reddit data includes sensitive discussions, ethical scraping is crucial. Here, I do scrape the userid from Reddit for the sake of personalized recommendations. However, we should always be aware of the potential data privacy issue and remove that from our scraping script.
        
        <br>
        Further, Reddit is a public platform, but it’s still best practice to inform users in any subsequent reporting that the data collected is public and gathered according to ethical standards. Thus, we may add a description saying how we utilized their data at the end of the personal recommendation message in the future.

    2. Bias in Sentiment Analysis and Recommendation
        Pre-trained models can carry biases, especially regarding sentiment or assumptions about medical preferences. Therefore, if possible, we should fine-tune the model to achieve better performance if we have access to better dataset. 
    
    3. Personalized Recommendation
        It is still unclear why precise recommendation fails: prompting methods, unrelated comments, noisy data... These hasn't been investigated due to the time constraint. Also, current method of using level-based recommendation might not be too ideal since the way it divides the group might introduce extra bias and is arbitary. More sophisticated methods should be further investiagated, leveraging the power of llm.




        





