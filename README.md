# Generativity
This repo hosts the required files, data and codes/scripts that were utilized for generativity project where we used deep learning to understand the generative component of human brain.

## Summary of The Project
Human cognitive and linguistic generativity depends on the ability to identify abstract relationships between perceptually dissimilar items. Marcus et. al. (1999) found that human infants can rapidly discover and generalize patterns of syllable repetition (reduplication) that depend on the abstract property of identity, but simple recurrent neural networks (SRNs) could not. They interpreted these results as evidence that purely associative neural network models provide an inadequate framework for characterizing the fundamental generativity of human cognition. Here, we present a series of deep long short-term memory (LSTM) models that identify abstract syllable repetition patterns and words based on training with cochleagrams that represent auditory stimuli. We demonstrate that models trained to identify individual syllable trigram words and models trained to identify reduplication patterns discover representations that support classification of abstract repetition patterns. Simulations examined the effects of training categories (words vs. patterns) and pretraining to identify syllables, on the development of hidden node representations that support repetition pattern discrimination. Representational similarity analyses (RSA) comparing patterns of regional brain activity based on MRI-constrained MEG/EEG data to patterns of hidden node activation elicited by the same stimuli showed a significant correlation between brain activity localized in primarily posterior temporal regions and representations discovered by the models. The results show that with pretraining to discriminate between individual syllables, deep associative models can achieve above-chance performance on this critical example of human cognitive generativity by drawing on discoverable representations similar to those employed in human neural processing.

## Hardware, and Software
Simulations were conducted on a Linux workstation with an Intel(R) Xeon(R) Gold 5218 CPU @ 2.30GHz, 98-gb of RAM, and an NVIDIA Quadro RTX 8000 (48-gb) graphics card. Simulations were conducted using Python 3.6, TensorFlow 2.2.0, and Keras 2.4.3. Each model requires approximately 48 hours to train on this workstation. This repository provides an up-to-date container with all necessary explanations and jupyter notebooks for running our training code and analyses.

## Data
### Training Data
We used the same audio files as in Gow et al. (2022). There was a total of 23 syllables, and we used sixteen in training (/ba/, /ÙI/, /dI/, /ÃI/, /ka/, /nI/, /pI/, /rI/, /Sa/, /sI/, /ta/, /DI/, /Tu/, /va/, /zI/, /Zu/) and seven in test (/fu/, /ga/, /hI/, /la/, /mI/, /wa/ and /ji/). Training data included 720 (240 for each pattern) phonemically balanced trisyllabic CV.CV.CV nonwords which were
created by concatenation of sixteen different syllables following the syllable reduplication patterns: ABA (e.g., as in ba-chih-ba), AAB (e.g., as in ba-ba-chih) and ABB (e.g., as in ba-chih-chih).
Testing data included 126 (42 for each pattern) phonemically balanced trisyllabic nonwords which were created in the same way. The auditory stimuli were recorded at a sampling rate of 44.1 kHz
with 16-bit sound quality and the duration of syllables was equalized to 250 ms (750 ms for each CV.CV.CV nonword). The input to the network was jittered cochleagrams of each auditory file.
A cochleagram is a spectrotemporal representation of auditory signal designed to mimic cochlear frequency decomposition. To create a cochleagram, we first removed any surrounding silence from
the audio files, and then passed each sound clip through a bank of 203 bandpass filters that were zero-phase, with varying center frequencies. Low-pass and high-pass filters were included to perfectly
tile the spectrum, resulting in a total of 211 filters. The final cochleagram representation was 150 x 211 (time x frequency) [Kell et al., 2018, Feather et al., 2019]. We generated the cochleagrams using Python with the numpy, scipy, and librosa libraries [Oliphant, 2007, McFee et al., 2015, Harris et al., 2020]. We then created ten jittered cochleagrams for each original cochleagram by utilizing data augmentation (specifically jittering in the time domain using random sigma values between (0.03, 0.09) [Um et al., 2017].

### Models
To model variable representation in the brain, we employed LSTMs to capture the temporal structure of auditory speech data. LSTMs are a type of recurrent neural network that are capable of retaining
past inputs and outputs for an extended period, making them well-suited for processing sequential data, such as time series and natural language. Based on the work of Avcu et al. [2023] and Magnuson
et al. [2020], we posit that LSTMs are a superior choice for capturing long-term dependencies in auditory speech data. The pretraining model consisted of a single LSTM layer with 512 nodes and a
dense layer with 23 nodes and softmax activation function. We used categorical cross-entropy as the loss function and ADAM (Adaptive Moment Estimation) [Kingma and Ba, 2014] optimization with a
fixed learning rate of 0.00001. The model was trained for 5000 epochs producing very high training and validation accuracy (over 90%).

The non-pretrained word and pattern learner models consisted of seven layers with 128, 256, 512, 1024, 512, 256 and 128 LSTM nodes respectively. On top of the LSTM layers, a dense layer with
vector outputs (69 for the word and 100 for the pattern learner networks). After every LSTM layer, we used a dropout layer with 0.85 (following Prickett et al. [2022]). Dropout is a regularization
method that helps generalization by forcing the model to make predictions that do not overly depend on any single feature, thus encouraging robustness and preventing overfitting. See Fig.1B. for the
structure of the main networks. The word and pattern learner models with pretraining consisted of the same architecture except for an additional input LSTM layer with 512 nodes with preloaded weights
coming from the pretraining. The cochleagrams of size 150 x 211 were fed into the first LSTM layer. Subsequently, the output of this layer was passed onto other layers respectively. The final layer was a dense layer that transformed the input vector X to an output vector Y of length n, where n represents the number of target classes (69 or 100). We employed the sigmoid activation function for the output layer, which returns a value between 0 and 1 and is centered around 0.5. Mean squared error loss was employed to calculate the mean of squares of errors between labels and predictions, with a batch size of 100. For optimization during training, we utilized ADAM as explained above. Each of the 720 words had ten jittered tokens, and seven of these tokens were utilized for training, while three were used for validation. For the pretraining, each syllable had two hundred tokens of which 180 were used for training and 20 were used for validation. Furthermore, the word and pattern learner networks were trained for 10,000 epochs, which involved complete iterations over the training set. The training parameters, such as the learning rate, the optimization algorithm, the loss function, etc., were adopted from Avcu et al. [2023].

### RSA Methods
Representational similarity analysis (RSA) involves assessing the correlation between decoding accuracy, determined by SVMs applied to ROI activation vectors in the brain (comprising 8 MNE
measures per ROI per timepoint), and SVMs applied to activation vectors derived from each of the 7 model layers. It is important to note that SVM accuracy functions as a measure of dissimilarity,
with high accuracy in two pairwise comparisons signifying a high level of dissimilarity between the compared items. Once we have both the neural and model decoding accuracy in the same shape
of vectors (see Supp materials for the detailed overview and schematic representation), we used Spearman’s rank correlation coefficient (rho), a nonparametric rank correlation measure, to assess the
similarity between the decoding accuracy vector of the model and that of the brain. To enhance the reliability of our results, we employed the Monte Carlo permutation test. This simulation technique
allowed us to evaluate the likelihood of obtaining the observed correlation by chance, considering the variability in our data. It offers a valuable means of verifying result robustness and gaining insight into the uncertainty associated with the correlation coefficient. The p-values associated with each correlation coefficient are based on 10,000 permutations. Upon completing this procedure, we generated a matrix of dimensions 68x21 for each model, which contained correlation coefficients for every pairwise comparison across each layer (3x7). For visualization purposes, we aggregated decoding accuracy across pairwise comparisons by calculating the average of the rho values, transforming the 68x21 matrix into a 68x7 format. Since p-values cannot be averaged, we adopt a criterion where we classify a layer as "non-significant" if any p-value for a pairwise comparison within that layer exceeds 0.05. For instance, in layer 1, if the p-values are as follows: 1vs2=0.001, 1vs3=0.06, 1vs2=0.0001, we would consider layer 1 as non-significant due to the second comparison (1vs3) having a p-value of 0.06. Subsequently, we reconstructed a p-value table, designating insignificant layers with 0.1 and significant ones with 0.01. This new p-table was used for masking the insignificant correlations in the RSA plots. Finally, to compare the mean correlation values of decoding vs. non-decoding ROIs across the seven layers of each model, we used Welch’s t-test (the unequal variances t-test).

## Results
### Model Accuracies
In our experimental setup, both a word learner, exposed to a corpus of 720 distinct words, and a pattern learner, designed to acquire three specific patterns, underwent training in two scenarios:
one with pretrained weights and the other without. The results, as illustrated in Fig.2, reveal significant disparities in their learning trajectories. In the absence of pretrained weights, both learners encountered challenges in achieving satisfactory performance levels over the 10,000 epochs. The pattern learner consistently maintained an average cosine similarity of around 0.34 throughout the entire training duration, encompassing training, validation, and test datasets. The word learner also remained relatively consistent, exhibiting a mean average cosine similarity of approximately 0.22 for training and validation accuracy (please note that test accuracy was not assessed for the word learner, given the uniqueness of each word). The pattern learner’s performance remained close to
chance, while the word learner’s performance, although better than chance, remained suboptimal for a successful model. In stark contrast, when pretrained weights were utilized, both learners reached
high-performance levels by the conclusion of the 10,000 epochs. The pattern learner, in particular, demonstrated an average cosine similarity of 0.71 for training, 0.59 for validation, and 0.56 for
the test dataset. Notably, the assessment of test data accuracy is pivotal, as it reflects the model’s performance on novel data. The word learner also excelled, achieving average cosine similarities of 0.72 for training and 0.67 for validation data. These outcomes underscore the considerable impact of  pretrained weights on the learning capabilities of our models.

### RSA Results
Decoding accuracy results (see TableS1) demonstrated that pretraining significantly boosted the decoding accuracy of both models and the progression of decoding accuracy between layers was different for word and pattern learner networks. In addition to the decoding analysis, we conducted a comparison of the decoding accuracy by time vectors extracted from the hidden unit activations of each layer within our models with neural activity derived from the 68 distinct ROIs. Our primary objective was to elucidate the close correspondence between human neural data and model performance in
relation to pretraining and task-specific capabilities. The findings, depicted in Fig.3, demonstrated that both the pattern and word learner models without pretraining exhibited moderate positive correlations with the neural data, particularly in the third layer of both model architectures. Notably, the regions of interest (ROIs) displaying these correlations included L_MTG-5, R_ITG-2, and R_STG-4 for the pattern learner, and L_ITG-1, L_MTG-5, L_postCG-1, L_STG-1, R_ITG-2 and 3, R_MTG-2, R_STG-1, R_STG-4, and R_STS-1 for the word learner. While none of the ROIs demonstrating moderate correlations with the pattern learner were decoder ROIs reported in Gow et al. [2023], it’s noteworthy that three of the ROIs showing moderate correlation with the word learner functioned as decoders, suggested to store reduplication patterns. In the case of models with pretraining, the outcomes reveal remarkably distinct patterns of correlations. Notably, the majority of decoder ROIs
(with the exception of R_STG-3) and several others, demonstrated notably high correlations with the pattern learner, particularly in the later layers, while the first layer did not show any significant correlation. Conversely, for the word learner, we observed a contrasting trend, wherein all decoder ROIs and numerous additional regions exhibited substantial correlations primarily with the initial layers, while the final layer displayed comparatively weaker correlations.

## Conclusion
Results suggest that associative mechanisms operating over discoverable representations capturing abstract stimulus properties account for a critical example of human cognitive 388 generativity highlighting the crucial significance of generative AI models in simulating and under standing cognitive generativity within the realms of human learning and representation.



