[![Freezing_of_Gait](https://img.youtube.com/vi/3-wrNhyVTNE/0.jpg)](https://www.youtube.com/watch?v=3-wrNhyVTNE&ab_channel=TheLancet)
# How to Detect Freezing of Gait Events for Parkinson's Patients?

*Parkinson's Disease is a brain disorder that causes unintended and uncontrollable movements, such as shaking, stiffness, and difficulty with balance and coordination. It affects 7 to 10 million Americans. Many Parkinson's patients suffer from Freezing of Gait(FOG). Freezing of Gait events causes a patient's feet to feel “glued” to the ground, preventing them from moving forward despite their attempts. **Please refer to the video above for an example.** Quality of life is negatively impacted by these events, particularly increased risk of falling, potentially leaving Parkinson's Disease patients confined to a wheelchair and unable to live independently. While researchers have multiple theories to explain when, why, and in whom FOG occurs, there is still no clear understanding of its causes. The purpose of this project is to help detect these FOG events to allow researchers to better understand them.*

This is part of a Kaggle competition. The goal of this competition is to detect freezing of gait (FOG) using data collected from a wearable 3D lower back sensor. From the [competition website](https://www.kaggle.com/competitions/tlvmc-parkinsons-freezing-gait-prediction/):
> "There are many methods of evaluating FOG, though most involve FOG-provoking protocols. People with FOG are filmed while performing certain tasks that are likely to increase its occurrence. Experts then review the video to score each frame, indicating when FOG occurred. While scoring in this manner is relatively reliable and sensitive, it is extremely time-consuming and requires specific expertise. Another method involves augmenting FOG-provoking testing with wearable devices. With more sensors, the detection of FOG becomes easier, however, compliance and usability may be reduced. Therefore, a combination of these two methods may be the best approach. When combined with machine learning methods, the accuracy of detecting FOG from a lower back accelerometer is relatively high. However, the datasets used to train and test these algorithms have been relatively small and generalizability is limited to date. Furthermore, the emphasis has been on achieving high levels of accuracy, while precision, for example, has largely been ignored. <br>
> Competition host, the Center for the Study of Movement, Cognition, and Mobility (CMCM), Neurological Institute, Tel Aviv Sourasky Medical Center, aims to improve the personalized treatment of age-related movement, cognition, and mobility disorders and to alleviate the associated burden. They leverage a combination of clinical, engineering, and neuroscience expertise to: 
> 1. Gain new understandings into the physiologic and pathophysiologic mechanisms that contribute to cognitive and motor function, the factors that influence these functions, and their changes with aging and disease (e.g., Parkinson’s disease, Alzheimer’s).
> 2. Develop new methods and tools for the early detection and tracking of cognitive and motor decline. A major focus is on leveraging wearable devices and digital technologies.
> 3. Develop and evaluate novel methods for the prevention and treatment of gait, falls, and cognitive function."
## 1. Data

For this project, I used the [Kaggle Dataset](https://www.kaggle.com/competitions/tlvmc-parkinsons-freezing-gait-prediction/) set up by the The Michael J. Fox Foundation. This dataset is broken up into several parts:<br>
***[Kaggle Wearable Sensor Data](https://www.kaggle.com/competitions/tlvmc-parkinsons-freezing-gait-prediction/data)***
>- The **tDCS FOG (tdcsfog) dataset**, comprising data series collected in the lab, as subjects completed a FOG-provoking protocol.
>- The **DeFOG (defog) dataset**, comprising data series collected in the subject's home, as subjects completed a FOG-provoking protocol
>- The **Daily Living (daily) dataset**, comprising one week of continuous 24/7 recordings from sixty-five subjects. Forty-five subjects exhibit FOG symptoms and also have series in the defog dataset, while the other twenty subjects do not exhibit FOG symptoms and do not have series elsewhere in the data.

This data is `AccV`, `AccML`, and `AccAP` Acceleration from a lower-back sensor on three axes: V - vertical, ML - mediolateral, AP - anteroposterior. The data also includes some patient information, including `Age`, `Sex`, alongside disease progression data like `Unified Parkinson's Disease Rating Scale`.
## 2. Methods

There are several steps to approach this problem of how to determine which features would increase negotiated rates and which one is best for hospitals to focus on. This is kept open ended and required going back and forth between steps:

1. **Exploratory Data Analysis:** This is key. It is important to a look at this data to determine how to proceed with future steps. What triggers events? What are the difference that can be identified between `StartHesitation`, `Walking`, or `Turning`. Does the patient information matter. What is the best way to create features and what models might be useful for this problem? Should we combine datasets into one model? These are just some of the insights I hope to get from this section.

2. **Feature Creature/Feature Selection:** Creating features based off the data will be important. This is a lot of data we are working with, and even though Kaggle provides GPU/TPUs for use, it will be about creating useful features, while also using a limiting the amount of them for whichever model we choose.

3. **Build Models:**  Finally we will create models using TensorFlow/Keras and other python libraries. The goal of the model is to classify events, so this will be a classification question, and we will use accuracy as our metric. Since the data has labelled and unlabeled data, we have to decide if unsupervised/semi-supervised model is a worthwhile pursuit.

## 3. Data Cleaning/Wrangling

* **Problem 1:** Each file for a patient is a different size length. **Solution:** Our model will pad files as we feed them in batches.

* **Problem 2:** The files are large and numerous. **Solution:** We will utilize GCP's GPU available from the Kaggle.

* **Problem 3:** There are unlabeled files. **Solution:** I was able to read community posts of how unsupervised learning was not working well for numerous people, so I decided to ignore this dataset.

## 4. EDA

[EDA Notebook](https://github.com/tennisvs/Springboard/blob/765d752ffd4bf0befd4e3c4b471d852d1824ff57/Freezing_Gait/1-eda-parkinsons-freezing-gait.ipynb)

There are many avenues I explored within EDA. However, the main goal was to look at events and determine the following:
- Is there a pattern to these labelled events? Yes, I noticed most events began and ended with great deltas increases and decreases in each accelerometer.
- How useful was the patient information? Much of the patient information was highly correlated with one another, but not necessarily the event itself.
- How "messy" was the accelerometer data? It seems like the data needed to be normalized.
- Can I combine the at home readings with the readings done at the doctors? The data seemed quite different, so I decided to separate the data for model building.
<img src="Pictures\Accelerometer_Measurements.png"  width="60%" height="30%">

## 5. Algorithms & Machine Learning

[Model Submission](https://github.com/tennisvs/Springboard/blob/765d752ffd4bf0befd4e3c4b471d852d1824ff57/Freezing_Gait/2-submission-freezing-gait.ipynb)

Feature Creation/Selection: In the end, the final model used normalized the readings for each accelerometer input as part of the model. I limited the features by not including any patient data. I did try rolling windows in previous models, alongside aggregate data, but this did not seem to have much effect.

I chose to work with the Python [sklearn](https://scikit-learn.org/) library and [TensorFlow](https://www.tensorflow.org/) for training my creating my model. In addition, I took a lot of inspiration from the work of [Baurzhan Urazalinov](https://www.kaggle.com/baurzhanurazalinov). You can find his work on Kaggle. He was the ultimate winner of the competition. I created two seperate models, one for the `Tdfog Dataset` and another for the `Defog Dataset`.The models are a combination of transformer encoder, which you can read about [here](https://arxiv.org/pdf/1706.03762.pdf) and bi-directional LSTMs. The reasoning behind choosing this, based off EDTA and me trying simple models is as follows:
- **Reason for LSTMs**: Based of EDTA, it becomes evident that there is a clear spike in a certain value and that triggers a fog event. LSTMs made sense for this. In addition, it came to my attention that the event also was stopped at future point in the future of the reading, and hence why bidirectional LSTMs were used. However, these are memory/computationally expensive, and had to rely on others for the code here.
- ***Reason for Transformer***: this is the part of model I relied on help from others in the Kaggle community
>"a model architecture eschewing recurrence and instead relying entirely on an attention mechanism to draw global dependencies between input and output.
The Transformer allows for significantly more parallelization and can reach a new state of the art in
translation quality after being trained for as little as twelve hours on eight P100 GPUs." - [Attention Is All You Need](https://arxiv.org/pdf/1706.03762.pdf)
- Limited the data, I did not use any of the subject data, or medication data. The community stated that this data had limited effect. In my simpler models, I tried combine a single directional LSTM with subject data, it had little impact.
- No features: outside of normalization, bidirectional LSTMs are memory and computationally intensive, limiting the resolution is key here. In simpler models I created, I was not performing nearly as well.
- During training, there is slight randomized rolling of positional encoding was done here, as done by the [Baurzhan Urazalinov](https://www.kaggle.com/baurzhanurazalinov). I decided to keep it here.

>***NOTE:** Since this was a classification problem, as stated in above, accuracy was the metric used. This submission had an accuracy of `0.410856`.
## 6. Future Improvements

* Different Models: Creating different models and averaging the scores is a method that [Baurzhan Urazalinov](https://www.kaggle.com/baurzhanurazalinov) utilized, and something I would have liked to use in the future. I was limited in time. In addition, I did not use the TPU and may have yielded faster results.
* Combining datasets: There might have been a way to label the `daily dataset` through an unsupervised method or even by just looking at the data myself. Though this would be time intensive, it might have yielded more data to train the models. 
* Creating a roadmap of the process: This is coming soon.

## 7. Credits
I would like to thank Springboard group and Branko Kovac, my mentor, for their help with this project. In addition, I wanted to thank [Baurzhan Urazalinov](https://www.kaggle.com/baurzhanurazalinov) for his guidance and work in this area. This was truly a great learning experience.


