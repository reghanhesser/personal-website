# Summary: 

In this assignment, I downloaded 10-Ks from the SEC edgar, cleaned the files, measured positve and negative sentiment from the text of those 10ks (as well as contextual sentiment measures for ESG, litigation, and risk metrics), downloaded returns around the 10-K release dates as well as brief time periods thereafter, converted those to CARs (aggregating the returns over a specific time period following the eventto capture the overall effect of the event on the stock price over time rather than just looking at individual daily returns), added return data to my sample, and conducted a formal analysis based on these findings.

While formulating my sentiment variables, I hypothesized that higher positive sentiment in 10-K filings corresponds to higher returns. Positive sentiment suggests investor trust, high future performance expectations, minimal past hindrances to the firms operations, and confidence both in investors and the company for its future. Positive legal, risk, and ESG metrics imply stability and adaptability to modern market expectations. However, I concluded 


# Data:
**MY SAMPLE:**
My sample comprises 10-K filings from S&P 500 companies for the year 2022. To assemble this sample, I created directories "inputs" and "10k_files" to store input files ('sp500_2022.csv') and download 10-K filings. Using the SEC Edgar downloader, I obtained the latest available 10-K filing for each company. I downloaded a list of S&P 500 companies for 2022 from Wikipedia and loaded them into a pandas dataframe named "sp500."

**CLEANING THE DATA:**

I used BeautifulSoup to parse the HTML content of the 10-K filing, removing any hidden XBRL elements. Additionally, I cleaned the text extracted from the HTML content by converting it to lowercase, removing non-alphanumeric characters, and collapsing multiple spaces into single spaces.

```# delete the hidden XBRL
for div in soup.find_all("div", {'style':'display:none'}):
div.decompose()

#clean data for parsing
soup_txt = soup.get_text().lower()
soup_txt = re.sub(r'\W',' ',soup_txt)
soup_txt = re.sub(r'\s+',' ',soup_txt)```


# Sentiment Analysis:

**NEAR_REGEX:**
The purpose of downloading NEAR_regex is to generate an expression pattern that captures the proximity of words within a certain distance or context. It enables algorithms to capture context, identify sentiment-bearing phrases and expressions, and adapt to language variability to allow for accurate assessments of sentiment in text data.

In the example below, I am creating a regular expression pattern that matches text where the pattern specified by legal and the pattern specified by BHR_positive_string are found within a certain maximum number of words between them to test positive sentiment around words in my "legal" string.

-Max_words_between=5 >>> This parameter specifies the maximum number of words allowed between the occurrences of the patterns specified by legal and BHR_positive_string. For my analysis, I kept this number as the baseline 5, because I was able to get adequate data doing so. I also did not want to have a number too high, leading to overly broad matches that include irrelevant text. Keeping it at 5 also implies that my words would have closer contextual relationships because of their close proximity.

-Partial=False >>> his parameter determines whether the pattern will match partial words or whole words only. In thus case, only whole words are permitted. This ensures precision and pattern specificity. 

**HOW DID I GENERATE POSITIVE/NEGATIVE SENTIMENT VARIABLES?:**

1) *LM Master Dictionary:* I used a .query to parse the master dictionary. .query allows me to filter rows simply based on a condition. For example, for rows where there is a number greater than 0 in the column "Negative", it indicaes that the word is correlated with negative sentiment. I join the lowercase words in the negative_words list to a single string separated by the "|" operator to represent all negative sentiment words from the LM dictionary. I repeated this for the positive words in the LM Master Dictionary.

```# Filter rows where 'negative' column is greater than 0
negative_words = df.query("Negative > 0")
negative_words = negative_words['Word'].copy()
negative_words = [elem.lower() for elem in negative_words]
LM_negative_string = "("+"|".join(negative_words) +")"
LM_neg = NEAR_regex([LM_negative_string])
```

2) *ML positive and ML negative lists:* I opened and read the two text files, and stored them in lists BHR_negative and BHR_positive, respectively. The words in each list are then joined using the '|' (OR) operator as before. 

These lines of code iterate through each string s in the list BHR_negative and removes any newline characters ('\n') using the replace() method. The resulting list is stored back in BHR_negative. The following line constructs a regular expression pattern string from the elements in the BHR_negative list. It joins the elements of BHR_negative using the '|' (or) separator and encloses them within parentheses () to create a regular expression pattern that matches any of the strings in BHR_negative. This method of formatting is necessary for using the NEAR_regex function. 

```with open('inputs/ML_positive_unigram.txt', 'r') as file:
    BHR_positive = file.readlines()
BHR_positive = [s.replace('\n', '') for s in BHR_positive]
BHR_positive_string = "("+"|".join(BHR_positive) +")"
BHR_pos = NEAR_regex([BHR_positive_string])
```
**The Following are the Number of Positive and Negative Sentiment Words In Each Dictionary:**
```print(len(BHR_positive))
print(len(BHR_negative))
print(len(negative_words))
print(len(positive_words))```
results:
75
94
2345
347

**HOW DID I GENERATE CONTEXTUAL SENTIMENT VARIABLES?:**


For this project, I focused on examining the contextual sentiment across three key areas: ESG (Environmental, Social, and Governance) factors, Legal Issues, and Risk. I integrated ESG analysis into this project as it aligns with my ongoing coursework in ESG finance (FIN 335)and our ongoing debate regarding whether firms actively engaged in ESG initiatives tend to yield higher returns.

Additionally, I chose to include Legal Issues as a metric because it provides valuable insights into a firm's stability and enables us to make informed predictions about its future profitability. Lawsuits, in particular, offer a glimpse into the legal landscape surrounding a company (compliance, disclosure transparency, reputationz) , which can significantly impact its operations and financial performance.

Furthermore, I explored Risks as a critical aspect of the analysis. While the adage "more risk equals more reward" is commonly heard, it's essential to delve deeper into how risks are perceived. Negative sentiment surrounding risks could indicate potential threats within the firm, affecting its financial condition and overall viability.

**BASIC SMELL TESTS:**
The topics I chose for my contectual sentiment analysis: ESG Metrics, Risk Metrics, and Legal Metrics all pass the "smell tests" as they all impact the financial performence and stability of investments. If positive sentiment is associeted with each of these metrics in a 10-k, favorable conditions and practices are shown to align with long tern investment goals. I checked to see if these measures were all correlated with high and low returns. I tested with existing information from my ESG Finance course (FIN335) to support my hypothesis that firms who engage in ESG practices and document them in company reports, see positive performance moreso than firms who do not. 

Another example to support my hypothesis that firms with many legal proceedings against them incur significantly more expenses, leading to reduced earnings (which are reflected in 10-k financial statements. The disclosure of legal liabilities may signal to investors that the firm is facing significant potential financial losses, which can contribute to negative market sentiment and stock price performance.

**INDUSTRIES AFFECTED:**

I do not believe that the metrics I chose are industry specific in the modern financial climate. However, there are industries that are affected more than others depending on the indicated measure. For example, firms in the energy sector have been put under scrutiny due to the negative externalities they produce- sucha as pollution and emissions. Technology firms face high legal and regulatory issued such as cybersecurity and intellectual property protection. Financial services face high risk metrics due to market volitility and the ever-changing geopoliticalclimate.

**POTENTIAL CAVEATS OF THESE SENTIMENT METRICS:**

Sample representativeness, data quality, and the subjectivity of sentiment analysis methods can introduce biases and limitations. Additionally, the scope of analysis may be limited, and other influential factors may not be captured. Contextual sentiment and its impact on firm performance may vary over time due to changes in market conditions, regulatory environments, stakeholder expectations, and other factors. While correlations between contextual sentiment and firm performance may be identified, establishing causation requires more extensive research. Overall, t's essential to interpret the findings cautiously, considering the limitations and caveats associated with the data and analysis methods used.

**WORD LISTS:**
```
legal = '(legal|compliance|regulatory|law|legislation|litigation|lawsuit|court|regulation)'
esg = '(sustainability|environmental|social|governance|ethical|diversity|equality|inclusion|corporate responsibility)'
risk = '(mitigation|opportunity|resilience|oversight|adaptability|manageable|prepared|uncertainty|vulnerability|exposure|hazard|threat|pitfall|downside|deterioration|instability)'
```

**FORMULA:**
```
Legal_positive_regex = NEAR_regex([legal, BHR_positive_string], max_words_between= 5, partial=False)
```
As previously mentioned, this generates an expression pattern that captures the proximity of words within a certain distance or context. In this case I am capturing positive sentiment using Machine Learning from legal text within each 10-K.

**USING SOUP and LOOPING THROUGH THE 10Ks:**

After processing the sentiment word lists, the code iterates through the S&P 500 companies' 10-K filings. Inside the loop, each 10-K filing's text data is extracted and stored as soup_txt. The document length (doc_length) is calculated by counting the words in the text. Sentiment analysis is then conducted by tallying the occurrences of positive and negative sentiment words using regular expressions. For the Loughran-McDonald (LM) dictionary, counts are stored in variables LM_positive_ and LM_negative_, while for the machine learning (ML) sentiment word lists, counts are stored in BHR_positive_ and BHR_negative_. Finally, the sentiment analysis results are printed, indicating the document length (doc_length) and the count of positive sentiment words (BHR_positive_).

Finally, sentiment scores are calculated by dividing the count of positive and negative sentiment words by the document length and multiplying by 100 to get percentages.

These sentiment scores are then stored in the sp500 DataFrame under columns such as 'Positive LM Sentiment', 'Negative LM Sentiment', 'Positive BHR Sentiment', and 'Negative BHR Sentiment', representing sentiment scores based on the LM dictionary and the machine learning sentiment word lists.

**Text Cleaning:**

soup_txt = soup.get_text().lower(): Extracts the text content from the HTML soup object and converts it to lowercase.
soup_txt = re.sub(r'\W',' ',soup_txt): Uses a regular expression to remove non-alphanumeric characters (symbols, punctuation, etc.) and replaces them with whitespace.
soup_txt = re.sub(r'\s+',' ',soup_txt): Uses a regular expression to replace multiple consecutive whitespace characters with a single whitespace.

**Counting Sentiment Occurrences:**

The code then proceeds to count the occurrences of various sentiment-related patterns within the cleaned text data. For example:
LM_positive_ = len(re.findall(LM_pos, soup_txt)): Counts the occurrences of positive sentiment patterns defined by the LM (Loughran-McDonald) dictionary.
Legal_positive = len(re.findall(Legal_positive_regex, soup_txt)): Counts the occurrences of positive sentiment patterns related to legal topics, using a regular expression pattern.

**Calculating Sentiment Scores:**

After counting the occurrences of sentiment-related patterns, the code calculates sentiment scores by dividing the count of each sentiment pattern by the total document length (number of words) and then multiplying by 100 to convert it into a percentage.
For example:

```
sp500.loc[index, 'Positive LM Sentiment'] = ((LM_positive_ / doc_length) * 100):

```
Calculates the percentage of positive sentiment words identified by the LM dictionary relative to the total document length and assigns it to the 'Positive LM Sentiment' column in the sp500 DataFrame.
Similar calculations are performed for negative sentiment scores and other sentiment categories.
In summary, this code snippet cleans the text data extracted from HTML documents and then counts the occurrences of various sentiment-related patterns within the text to calculate sentiment scores, which are then stored in the sp500 DataFrame for further analysis.

**LOOPING THROUGH EACH 10-K TO GET EACH FILING DATE AND APPENDING ACCESSION:**

Iterating through the sp500 dataframe by CIK, I accessed each firm's respective 10-K file. Constructing a path pattern (firm_folder) to search for 10-K files for the current CIK within the zip archive. If no files were found, it moved to the next CIK. If files were found, it selected the first match as the path to the file (fpath), assuming one 10-K filing per CIK. I extracted the accession number from the file path and added it to the sp500 DataFrame using the .split('/') function. 

*Understanding split('/'):* split turns the fpath string into a list of substrings based on the '/' character. This operation essentially breaks the file path into segments, where each segment is separated by a '/'.
After splitting the file path, the resulting list (split) contains the individual components of the file path. 
In the provided code, split[3] is then used to extract the fourth element of the split list, which corresponds to the accession number. This accession number is then added to the sp500 DataFrame under the 'Accession' column.

For each company, it constructs a unique URL that points to the SEC website's page for that company's 10-K filing.
After constructing the URL, it sends a request to the SEC website to fetch the webpage corresponding to the 10-K filing. (I have access through the HTTP session object that allows you to send HTTP requests and retrieve HTML content from web pages.)

```with zipfile.ZipFile('10k_files/10k_files.zip', 'r') as zipfolder:
    # Get list of files in zipped folder
    file_list = zipfolder.namelist()
    
    # Loop over the DataFrame containing CIKs
    for index, row in tqdm(sp500.head(20).iterrows(), total=len(sp500), desc="Processing CIKs"):
        cik = str(row['CIK'])  # Convert CIK to string

        # Get a list of possible files for this firm
        firm_folder = f"sec-edgar-filings/{cik.zfill(10)}/10-K/*/*.html"
        possible_files = fnmatch.filter(file_list, firm_folder)
        
        # If no files found, continue to the next firm
        if len(possible_files) == 0:
            continue
                
        # Select the first match as the path to the file
        fpath = possible_files[0]
        
        #.split to get * and cik
        split = fpath.split('/')
        
       #add to original dataframe
        accession = split[3]
        
        #add accession to sp500 dataframe .loc
        sp500.loc[index, 'Accession'] = accession
        
        # Open the file and read its contents
        with zipfolder.open(fpath) as report_file:
            html = report_file.read().decode(encoding="utf-8")
            
        soup = BeautifulSoup(html, "lxml-xml")
```

```HTMLSession()
session.headers.update({'User-Agent':'Reghan Hesser rlh224@lehigh.edu'})```

Once the webpage is fetched, it extracts the filing date from the specific HTML element using a CSS selector (div.formContent > div:nth-child(1) > div:nth-child(2)). This element contains the filing date information for every file. 
If the filing date is successfully extracted, it appends this date to the DataFrame sp500 under the column name 'Filing Date' for the corresponding company.
If the element containing the filing date is not found, it prints a message indicating that the filing date element was not found.
This process repeats for each company in the DataFrame sp500. 

**CREATING 3 VERSIONS OF BUY AND HOLD RETURN VARIABLES AROUND 10K-DATE**

1) RETURN ON DAY OF THE FILING (DAY T):
As mentioned earlier, using CRSP_2022.csv dataframe, (which gave me a list of tickers, dates, and the respective returns on those dates) I performed a merge with the sp500 dataframe. To do this, I needed to rename the "Symbol" column provided in the original sp500 dataset to 'ticker' and "Filing date" to "Fate" in order to perform the merge on these two variables. I also needed to convert the date cokumn to datetime type in the sp500 dataframe before merging. The merege allowed me to find the returns on the day of each 10-K filing.

```url = 'https://github.com/LeDataSciFi/data/raw/main/Stock%20Returns%20(CRSP)/crsp_2022_only.zip'

#CRSP = pd.read_stata(url)   
# <-- that code would work if I had uploaded the data as a csv file, but GH said it was too big to upload 
# so I zipped it. We need a little workaround to download it:

with urlopen(url) as request:
    data = BytesIO(request.read())

with ZipFile(data) as archive:
    with archive.open(archive.namelist()[0]) as stata:
        crsp = pd.read_stata(stata)
crsp
#merge on firm and date
import pandas as pd

# Merge the DataFrames based on 'ticker' and 'date'
# Convert 'date' column to datetime type in the sp500 DataFrame
sp500['date'] = pd.to_datetime(sp500['date'])

# Now, you can proceed with the merge
merged_df_date_t = pd.merge(sp500, crsp, on=['ticker', 'date'])

# Print the merged DataFrame
print(merged_df_date_t)
```

2) BUY AND HOLD RETURN OVER TIME SPAN T TO T+2

3) BUY AND HOLD RETURN OVER TIME SPAN T+3 TO T+1

**MY FINDINGS AND RESULTS:**
1) RETURN ON DAY OF THE FILING (DAY T): using individual scatterplots for each sentiment variable against returns. The scatterplots visualize the relationship between each sentiment variable and the returns of the stocks in my dataset. By plotting each sentiment variable separately, I examined how changes in sentiment correspond to changes in returns. The following are the relationships I was able to extract. I confirmed this further with the use of a heatmap.  

-The highest correlation was between Negative ESG sentiment and returns (.45), followed by Positive BHR sentiment and returns(.43), then Positive EGS sentiment and returns (.40)
-The lowest correlations were between retruns and Positive Risk Sentiment, Positive Legal Sentiment and returns, and Negative risk sentiment and returns. I am assuming the languege for my risk metrics was too ambiguous.

Overall the ML (BHR) positive and sentiment variables were in line with my hypothesis. Without using nearregex to align positive and negative sentiment to my personal metrics, the firms with positive ML sentiment alone also had higher returns. I was also suprised to see very little correlation between returns and positive/negative sentiment scores from the LM lists. 

The relationships between the returns variable and the two “LM Sentiment” variables (positive and negative) and the relationships between the returns variable and the two “ML Sentiment” variables (positive and negative) are different. The correlations from the BHR (ML) are some of the highest correlations out of those tested, and the LM lists are some of the lowest.

**What explains this lack of correlation?**
It is important for me to reiterate the caveats with sentiment analysis through machine learning. As stated in the ML_JFE, there are multiple positive unigrams that are preceeded by "not" that are negative bigrams. There are also negative bigrams, that when negated, are still being scored as negative by the algorithm. Differently put, It is true that negating positive words generates neg- ative bigrams, but it is also true that negating negative words generates negative bigrams. 
Also, as seen in figure 3 in ML_JFE, the ML algorithm strongly concentrates the bulk of the points in the upper center of the triangle, corresponding to unigrams that are mostly neutral, with some instances of explicit positive and negative context. This tells us that some tokens may need disambiguation (improve, confident) with further context outside of the words themselves. I beleieve the paremeters I set while using the regex function with the ML dictionary words allowed for closer precision as I places limitations on the words in between and did not allow for partial speech. 

**Concluding Notes**
The high correlations between Environmental, Social, and Governance (ESG) sentiments and returns underscore the significance of ESG inclusion in today's market landscape. ESG factors have increasingly become central to investment decisions and corporate strategies due to their potential impact on financial performance, risk management, and stakeholder value creation. Risk, along with is limited correlations between sentiment and returns implys that risk is subject to every firm and is volitle from year to year. Yes, risk is central to understanding performance, but a growth company will have different risk metrics than a mature one, and sentiment on such risks is extremely ambiguous. 

It can  be concluded that sentiment analyis is simply a mere tool in gauging market sentiment, investor sentiment, or sentiment towards specific entities such as companies or products. While these plots provide insights into the relationship between sentiment and returns, they do not necessarily establish causation or provide definitive conclusions.
