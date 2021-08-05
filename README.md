# Email Spam Classification

Many email services today provide spam filters that are able to classify emails into spam and non-spam email with high accuracy. I will be using SVMs to build my own spam filter. I will be training a classifier to classify whether a given email, , is spam `(y=1)` or non-spam `(y=0)`. In particular, I will convert each email into a feature vector ![](https://latex.codecogs.com/gif.latex?x%5Cin%5Cmathbb%7BR%7D%5En)

The dataset included for this exercise is based on a a subset of the [SpamAssassin Public Corpus](https://spamassassin.apache.org/old/publiccorpus/)


## Preprocessing emails

In processEmail.m, I have implemented the following email preprocessing and normalization steps:
- <b>Lower-casing:</b> The entire email is converted into lower case, so that captialization is ignored (e.g., IndIcaTE is treated the same as indicate).
Stripping HTML: All HTML tags are removed from the emails. Many emails often come with HTML formatting; we remove all the HTML tags, so that only the content remains.
- <b>Normalizing URLs:</b> All URLs are replaced with the text "httpaddr".
- <b>Normalizing Email Addresses:</b> All email addresses are replaced with the text "emailaddr".
- <b>Normalizing Numbers:</b> All numbers are replaced with the text 'number'.
- <b>Normalizing Dollars:</b> All dollar signs ($) are replaced with the text 'dollar'.
- <b>Word Stemming:</b> Words are reduced to their stemmed form. For example, 'discount', 'discounts', 'discounted' and 'discounting' are all replaced with 'discount'. Sometimes, the Stemmer actually strips off additional characters from the end, so 'include', 'includes', 'included', and 'including' are all replaced with 'includ'.
- <b>Removal of non-words:</b> Non-words and punctuation have been removed. All white spaces (tabs, newlines, spaces) have all been trimmed to a single space character.

### Vocabulary listing

After preprocessing the emails, I have a list of words for each email. The next step I choose which words I would like to use in my classifier and which I would want to leave out. For this, I have chosen only the most frequently occuring words as my set of words considered (the vocabulary list). The complete vocabulary list is in the file `vocab.txt`. Given the vocabulary list, I can now map each word in the preprocessed emails into a list of word indices that contains the index of the word in the vocabulary list. 
    In the code, I am giving a string `str` which is a single word from the processed email. I then look up the word in the vocabulary list `vocabList` and find if the word exists in the vocabulary list. If the word exists, I then add the index of the word into the word indices variable. If the word does not exist, and is therefore not in the vocabulary, I skip the word.
    The code below will run my code on the email sample
 ```
 %% Initialization
clear;

% Extract Features
file_contents = readFile('emailSample1.txt');
word_indices  = processEmail(file_contents);
% Print Stats
disp(word_indices)
```

## Extracting features from emails

You will now implement the feature extraction that converts each email into a vector in ![](https://latex.codecogs.com/gif.latex?%5Cmathbb%7BR%7D%5En). For this I will be using words in vocabulary list. Specically, the feature ![](https://latex.codecogs.com/gif.latex?x_i%5Cin%5C%7B0%2C%5C%3B1%5C%7D) for an email corresponds to whether the ![](https://latex.codecogs.com/gif.latex?i)-th word in the dictionary occurs in the email. That is, ![](https://latex.codecogs.com/gif.latex?x_i%20%3D%201) if the ![](https://latex.codecogs.com/gif.latex?i)-th word is in the email and ![](https://latex.codecogs.com/gif.latex?x_i%20%3D%200) if the ![](https://latex.codecogs.com/gif.latex?i)-th word is not present in the email.

The code below will will run your code on the email sample
```
% Extract Features
features = emailFeatures(word_indices);
```

## Training SVM for spam classification

I then loaded a preprocessed training dataset that will be used to train an SVM classifier. `spamTrain.mat` contains 4000 training examples of spam and non-spam email, while `spamTest.mat` contains 1000 test examples. Each original email is processed using the `processEmail` and `emailFeatures` functions. After loading the dataset, the code will proceed to train a SVM to classify between spam () and non-spam () emails

```
% Load the Spam Email dataset
% You will have X, y in your environment
load('spamTrain.mat');
C = 0.1;
model = svmTrain(X, y, C, @linearKernel);

p = svmPredict(model, X);
fprintf('Training Accuracy: %f\n', mean(double(p == y)) * 100);

% Load the test dataset
% You will have Xtest, ytest in your environment
load('spamTest.mat');

p = svmPredict(model, Xtest);
fprintf('Test Accuracy: %f\n', mean(double(p == ytest)) * 100);
```

## Top predictors for spam

To better understand how the spam classifier works, I inspected the parameters to see which words the classifier thinks are the most predictive of spam. The code below finds the parameters with the largest positive values in the classier and displays the corresponding words

```
% Sort the weights and obtin the vocabulary list
[weight, idx] = sort(model.w, 'descend');
vocabList = getVocabList();
for i = 1:15
    if i == 1
        fprintf('Top predictors of spam: \n');
    end
    fprintf('%-15s (%f) \n', vocabList{idx(i)}, weight(i));
end
```
