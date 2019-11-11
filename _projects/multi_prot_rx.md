---
layout: project
title: Multi-Protocol Receiver
nav: projects
importance: 100
description: a 10Gbps-1.6Gbps receiver supporting various ethernet, usb, and display port communication protocols
img: /assets/img/multi_prot_rx.svg
---
<div class="container-fluid p-0">
  <img class="img-responsive col-12" src="{{ '/assets/img/multi_prot_rx.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center" style="color: #78909c;"><u>Figure 1:</u> Overview of the receiver. My design contribution includes (a) Summmer, (b) Offset Correction Loop, (c) Samplers, (d) Data Paths(TSPC Flops), (e) Clock Tree, (f) Deserializer</h6>
</div>

<b>High Speed Serial Interfaces (HSSI)</b> are the backbone of modern communication systems. They take data from multiple slower lanes and convert them into a single high-speed serial lane in order to send the data through long channels. As the speed of these communication links increases, the signal suffers more attenuation while passing through a channel which may be a PCB, a coaxial cable, an optical fiber, or an inductively coupled link. The channel virtually follows a Low Pass Response, with occasional deviations because of impedance mismatches in the channel. For high-frequency signals, the channel loss is usually so high that the signal at the end of the channel becomes unreadable. It becomes of paramount importance to properly equalize the channel's response in order to have good detection. There are two primary techniques to equalize the channel response, one is <b>CTLE (Continuous Time Linear Equalizer)</b>, and the other one is <b>DFE (Decision Feedback Equalizer).</b>  

CTLE, as the name suggests, is a linear equalization scheme, and generates a high pass response nullifying the low pass response of the channel. It does a pretty good job of amplifying the signal, but the drawback is that it amplifies noise as well. Moreover, since the channel is not exactly Low Pass, a High Pass equalization is not enough to account for the deviations in the frequency response of the channel. Decision Feedback Equalizer is a non-linear equalization scheme that compensates for the dispersion that occurs in the channel. It also removes the noise in the analog signal by sampling it. More often than not, both CTLE and DFE are required to optimally equalize the channel's response.

In this particular receiver implementation shown in Figure 1, both CTLE and DFE are employed to compensate for the signal losses. The equalized data is sampled using a low-noise, offset calibrated comparator, which is itself a part of the DFE loop. After sampling, the data is deserialized into multiple lanes and handed-off to digital for running multiple calibrating sequences and parameter training to achieve low BER (Bit Error Rate).

The following sections explain the basic functioning of the sub-circuits of the Figure-1 receiver, along with the parameters they were designed for:


<h3 class="title mt-4 p-0 text-left">DFE Architecture</h3>

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/dfe.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 2:</u> A Typical Channel Response</h6>
</div>

Any channel can be characterized by its impulse response, an example of which can be seen in Figure 2. The difference between each dot on the x-axis represents the Data Unit Interval(UI). The impulse response shows how a particular input bit's power, after the channel, interferes with other bits. This phenomenon of one bit's power bleeding into the next bits is known as <b>ISI (Inter-Symbol Interference)</b> and is because of the bandwidth limitation of the channel. The actual bit is recognized by the highest point (main-sample point) in the impulse response, while the other tail values (Post-Cursors) are the unwanted interference of this bit with the next data bits. ISI is minimized by reconstructing the post-cursor values and adding/subtracting them appropriately from the original data that is sampled. This is called <b>Decision Feedback Equalization(DFE)</b> and can be used to minimize previous bits' interference energy from the present bit in order to achieve accurate digital detection.

A 1-Tap Speculative, 5-Tap DFE architecture(Figure 3) is implemented to effectively cancel the 5 post-cursor taps of the channel response(black in Figure 2) to achieve DFE equalized response(ideally; red in Figure 2). The DFE in concoction with the CTLE and VGA helps in efficient detection of 10Gbps data till 35dB loss channels. The first tap is implemented in an unrolled manner, using the reference branch of the slicer. All other tap information is fed-back by just delaying(resampling), and scaling the comparator output. The DFE follows the following equation:

$$ v_{dfe} = v_{in} \pm h_1 \pm h_2 \pm h_3 \pm h_4 \pm h_5 $$

where \\(v_{dfe}\\) is the DFE equalized signal which comes at the input of slicer, \\(v_{in}\\) is the input to the DFE system, and \\(h1\\),\\(h2\\), \\(h3\\), \\(h4\\), \\(h5\\) are the post-cursor tap magnitudes which are added or subtracted from \\(v_{in}\\) based on the previous detected bits (0 or 1). The critical paths in the DFE implementation warrant the design of very high-speed comparator, flip-flops(delaying elements), and summer.   


<div class="container-fluid p-0">
  <img class="img-responsive col-12" src="{{ '/assets/img/dfe_arch.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center" style="color: #78909c;"><u>Figure 3:</u> A 1-Tap Speculative, 5 Tap Decision Feedback Equalizer</h6>
</div>

<h3 class="title mt-4 p-0 text-left">Summer</h3>

Summer is just an amplifier, which adds or subtracts the tap values based on the previous bit information. The concept of equalization by addition/subtraction is applicable when the system is linear, hence, the linearity of the summer is an important design aspect. As shown in Figure 4, the input to the summer amplifier stage connects to one branch, while the feedback signals \\(S\\) (based on previous detected bits) from the samplers come to alternate amplifier branches, depending on whose values, a current of certain magnitude linearly adds or subtracts from the input signal. The resultant output \\(v_{dfe}\\) is the equalized signal which ultimately gets sampled by the slicer. A source degenerated highly linear summer amplifier stage is designed for extremely low output settling time to meet the DFE timing specifications.  

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/summer.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 4:</u> Summer</h6>
</div>


<h3 class="title mt-4 p-0 text-left">Samplers (Slicer/Comparator)</h3>
The Slicer is a critical part of the receiver system. It is the stage that converts a very low amplitude signal into streams of perfect 1s and 0s. The popular Strong-Arm Latch based dynamic comparator, is used for sampling the input eye, as shown in Figure 5. The output of the comparator is the most critical signal of the system, as all other on-chip procedures depend on the correctness of this data. Since the input to the comparator can have extremely low amplitudes, the non-idealities of the comparator make it highly susceptible to passing erroneous data. The comparator design is optimized for extremely low input-referred random and kickback noise, while the offset of the comparator is corrected by a highly accurate offset-cancellation system.


<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/comparator.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 5:</u> 1 bit Comparator (Sampler)</h6>
</div>


<h3 class="title mt-4 p-0 text-left">Current Steering DAC</h3>
The delay of the comparator increases as the input signal amplitude decreases. The input amplitude for which the comparator gives maximum allowed delay for proper functioning at a given frequency is called the sensitivity of the comparator. The input vertical eye-opening must be greater than or equal to the sensitivity of the comparator for correct detection of data. If the noise and offset of the comparator cause the perceived eye-opening to be smaller than the sensitivity, it will cause erroneous detection. A DAC is used to calibrate the offset of the comparator using a reference branch to minimize the offset's effect on input eye-opening. During the calibrating sequence, a slow ramp(code sweep) is given to the DAC connected to the reference branch to see the code for which the output of the comparator flips, and that code is then set at the reference branch for mission mode of the chip. The Current Steering DAC used provides many benefits including fast and glitch-free transient response, low ground bounce, and natural differential outputs required for the comparator's reference branch. The DAC is fully controlled using Thermometric logic, which ensures extremely-low Differential and Integral Non-Linearities (<1LSB).

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/current_steering_dac.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 6:</u> N Bit Thermometric Current Steering DAC</h6>
</div>


<h3 class="title mt-4 p-0 text-left">TSPC Flip-Flops</h3>
Designing high-speed, low-power flip-flops is crucial since they are omnipresent in transceiver systems. Understanding how to get desired setup time, hold time, and clock-to-q delay with minimum possible power consumption can help meet crucial power budgets. I have written a separate article explaining and stressing on these trade-offs for a TPSC flop <a href="https://yatingilhotra.github.io/projects/tspc/" target="_blank"> <b>here</b> </a>.

<h3 class="title mt-4 p-0 text-left">Clock Tree</h3>

The maximum frequency of the clock on which a system can run is limited by the latency of individual elements. For example, the maximum frequency of clock, \\(f_{clk}\\) (minimum Time Period \\(T_{min}\\)) for a two-flop system, as shown in Figure 7, is decided by the time \\(t_1\\), the clock-to-q delay of the flop, \\(t_2\\), latency of the logic, and \\(t_{setup}\\), the setup time of the flop. For reliable operation, the two flop system must follow this relation:

$$ {\bf t}_1 + {\bf t}_2 + {\bf t}_{setup} < {\bf T_{min}}$$

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/clock_system.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 7:</u> A two flop system</h6>
</div>

This is when we consider the clocks to be ideal. If we also account for the non-idealities of the clocks like the phase to phase mismatch among different phases of the clocks, and random/deterministic jitter on the clocks, the minimum time period equation becomes:

$$ {\bf t}_1 + {\bf t}_2 + {\bf t}_{setup} + {\bf t}_{skew} + {\bf t}_{jitter} < {\bf T_{min}},$$

If these non-idealities are not in check, they lead either to a slower communication system, or higher Bit Error Rates in a faster communication system. Therefore, it is critical to budget these parameters, and design for these parameters while implementing the clock tree. This is taken care of by designing sufficiently big sized CMOS clock buffers so that there is less mismatch between multiple instances, and also by meticulous, and symmetric layout strategies to distribute these clocks.



{%comment%}

The sampling procedure inherently cleans up noise, jitter, and waveform distortions present in the data.

For example, we can equalize the first post cursor by sampling the data, scaling it by a factor of \\(h_1\\), and then subtracting it from the next bit. Actually this can be done in two ways, popularly known as Rolled and Unrolled DFE implementations, as shown in Figure 3. Unrolled DFE implementation gives an advantage of delay of one operation, and hence makes it easier to meet the 1UI feedback timing requirement.

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/dfe_type.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 3:</u> Types of DFE Implementstions: (a) Rolled DFE  (b) Unrolled(Speculative) DFE</h6>
</div>

An extremely important aspect of the comparator design is its sensitivity, which is defined as the input amplitude for which the comparator gives sufficiently low delay.  


It uses the concept of regeneration to achieve large output swings in a very short amount of time.
It is important because the comparator is used in 1UI or 1.5UI paths(UI: Unit Interval, which is 100ps for a receiver supporting 10Gbps of data speeds) depending on the type of DFE implemented, and the delay of the strong-arm comparator increases as the input amplitude decreases. Moreover, the input amplitudes for high loss channels can come out to be very small values(of the order of a few tens of mVs) even after boosting and equalizing it. That's why the sensitivity parameter is so critical. Along with the delay of the comparator, the input-referred noise and the offset of the comparator also directly add up to the sensitivity number. Hence, it is important to ensure a low noise and offset numbers for the comparator.    

By using fully thermometrically controlled DAC, following Pelgrom's Law, and appropriate layout strategies, less than 1LSB (Least Significant Bit) INL and DNL was achieved. The LSB was kept significantly small to precisely cancel the comparator offset.    

It is also defined by it's input-referred noise and it's offset. For systems where you can't afford big sensitivity numbers, it becomes essential to correct the offset of the comparator. That is done by calibrating the reference branch of the comparator(Figure 6) using a DAC.

{%endcomment%}



{% comment %}

During my PhD I worked on a couple of interesting projects related to machine translation. In this page I plan to talk about two of them: one which introduced the idea of <i>contextual parameter generation (CPG)</i> and one which proposes a novel framework for curriculum learning and applies it to the problem of machine translation. The following post is currently about the first project, but it will soon be updated with information about the second.

<div class="alert alert-danger" role="alert">
  <b>NOTE:</b> The following article has also appeared on the <a href="https://blog.ml.cmu.edu/2019/01/14/contextual-parameter-generation-for-universal-neural-machine-translation/" target="_blank">CMU machine learning department blog</a>.
</div>

<div class="container-fluid p-0">
  <img class="img-responsive col-12" src="{{ '/assets/img/multi_prot_rx.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center" style="color: #78909c;"><u>Figure 1:</u> Overview of the contextual parameter generator that is introduced in this post. The top part of the figure shows a typical neural machine translation system (consisting of an encoder and a decoder network). The bottom part, shown in red, shows our parameter generator component.</h6>
</div>

<div class="col mt-4 p-0">
  Machine translation is the problem of translating sentences from some source language to a target language. Neural machine translation (NMT) directly models the mapping of a source language to a target language without any need for training or tuning any component of the system separately. This has led to a rapid progress in NMT and its successful adoption in many large-scale settings.
</div>

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/projects/machine_translation/1_mt.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 2:</u> Illustration of the machine translation problem.</h6>
</div>

<div class="col mt-4 p-0">
  NMT systems typically consist of two main components: the encoder and the decoder. The encoder takes as input the source language sentence and generates some latent representation for it (e.g., a fixed-size vector). The decoder then takes that latent representation as input and generates a sentence in the target language. The sentence generation is often done in an autoregressive manner (i.e., words are generated one-by-one in a left-to-right manner).
</div>

<h3 class="title mt-4 p-0 text-left">Multilingual Machine Translation</h3>

<div class="col mt-4 p-0">
  It is easy to see how this kind of architecture can be used to translate from a single source language to a single target language. However, translating between arbitrary pairs of languages is not as simple. This problem is referred to as <i>multilingual machine translation</i>, and it is the problem we tackle. Currently, there exist three approaches for multilingual NMT.
</div>

<div class="container-fluid text-center mt-4 p-0">
  <div class="row m-0">
    <div class="col-12 col-sm-4 mb-3 mb-sm-0">
      <img class="img-responsive" style="width: 100%;" src="{{ '/assets/img/projects/machine_translation/2_pairwise.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
    </div>
    <div class="col-12 col-sm-4 mb-3 mb-sm-0">
      <img class="img-responsive" style="width: 100%;" src="{{ '/assets/img/projects/machine_translation/3_universal.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
    </div>
    <div class="col-12 col-sm-4 mb-1 mb-sm-0">
      <img class="img-responsive" style="width: 100%;" src="{{ '/assets/img/projects/machine_translation/4_per_language.svg' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
    </div>
  </div>
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 3:</u> Overview of existing approaches for multilingual neural machine translation.</h6>
</div>

<div class="col mt-4 p-0">
  Assuming that we have \(L\) languages and \(P\) trainable parameters per NMT model (these could be, for example, the weights and biases of a recurrent neural network) that translates from a single source language to a single target language, we have:
</div>

<ol class="mt-2">
  <li>
    <b>Pairwise:</b> Use a pairwise NMT model per language pair. This results in \(L^2\) separate models, each with \(P\) parameters. The main issue with this approach is that no information is shared between different languages, and even between models translating from English to other languages, for example. This is especially problematic for low-resource languages, where we have very little training data available, and would ideally want to leverage the high availability of data for other languages, such as English.
  </li>
  <li>
    <b>Universal:</b> Use a single NMT model for all language pairs. In this case, an artificial word can be added to the source sentence denoting the target language for the translation. This results in a single model with \(P\) parameters, since the whole model is shared across all languages. This approach can result in overfitting to the high-resource languages. It can also limit the expressivity of the translation model for languages that are more "different" (e.g., for Turkish, if training using Italian, Spanish, Romanian, and Turkish). Ideally, we want something in-between pairwise and universal models.
  </li>
  <li>
    <b>Per-Language Encoder/Decoder:</b> Use a separate encoder and a separate decoder for each language. This results in a total of \(LP\) parameters and lies somewhere between the pairwise and the universal approaches. However, no information can be shared between languages that are similar (e.g., Italian and Spanish).
  </li>
</ol>

<div class="col mt-2 p-0">
  At <a href="http://aclweb.org/anthology/D18-1039" target="_blank">EMNLP 2018</a>, we introduced the <i>contextual parameter generator (CPG)</i>, a new way to share information across different languages that generalizes the above three approaches, while mitigating their issues and allowing explicit control over the amount of sharing.
</div>

<h3 class="title mt-4 p-0 text-left">Contextual Parameter Generation</h3>

<div class="col mt-4 p-0">
  Let us denote the source language for a given sentence pair by \(\ell_s\) and the target language by \(\ell_t\). When using the contextual parameter generator, the parameters of the encoder are defined as \(\theta^{(enc)}\triangleq g^{(enc)}({\bf l}_s)\), for some function \(g^{(enc)}\), where \({\bf l}_s\) denotes a language embedding for the source language \(\ell_s\). Similarly, the parameters of the decoder are defined as \(\theta^{(dec)}\triangleq g^{(dec)}({\bf l}_t)\) for some function \(g^{(dec)}\), where \({\bf l}_t\) denotes a language embedding for the target language \(\ell_t\). Our general formulation does not impose any constraints on the functional form of \(g^{(enc)}\) and \(g^{(dec)}\). In this case, you can think of the source language, \(\ell_s\), as a context for the encoder. The parameters of the encoder depend on its context, but its architecture is common across all contexts. We can make a similar argument for the decoder, and that is where the name of this parameter generator comes from. We can even go a step further and have a parameter generator that defines \(\theta^{(enc)}\triangleq g^{(enc)}({\bf l}_s, {\bf l}_t)\) and \(\theta^{(dec)}\triangleq g^{(dec)}({\bf l}_s, {\bf l}_t)\), thus coupling the encoding and decoding stages for a given language pair. This would make the model more expressive and could thus work better in cases where large amounts of training data are available. In our experiments we stick to the previous, <i>decoupled</i>, form, because it has the potential to lead to a common representation among languages, also known as an <i>interlingua</i>. An overview diagram of how the contextual parameter generator fits in the architecture of NMT models is shown in Figure 1 in the beginning of this post.
</div>

<div class="col mt-2 p-0">
  Concretely, because the encoding and decoding stages are decoupled, the encoder is not aware of the target language while generating it, and so, we can take an encoded intermediate representation of a sentence and translate it to any target language. This is because the intermediate representation is independent of any target language. This makes for a stronger argument that the intermediate representation produced by our encoder could be approaching a universal interlingua, more so than methods that are aware of the target language when they perform encoding.
</div>

<h4 class="title mt-4 p-0 text-left">Parameter Generator Network</h4>

<div class="col mt-4 p-0">
  We refer to the functions \(g^{(enc)}\) and \(g^{(dec)}\) as <i>parameter generator networks</i>. A simple form that works, and for which we can reason about, is to define the parameter generator networks as simple linear transforms:
  $$g^{(enc)}({\bf l}_s) \triangleq {\bf W^{(enc)}} {\bf l}_s,$$
  $$g^{(dec)}({\bf l}_t) \triangleq {\bf W^{(dec)}} {\bf l}_t,$$
  where \({\bf l}_s, {\bf l}_t \in \mathbb{R}^M\), \({\bf W^{(enc)}} \in \mathbb{R}^{P^{(enc)} \times M}\), \({\bf W^{(dec)}} \in \mathbb{R}^{P^{(dec)} \times M}\), \(M\) is the language embedding size, \(P^{(enc)}\) is the number of parameters of the encoder, and \(P^{(dec)}\) is the number of parameters of the decoder.
</div>

<div class="col mt-2 p-0">
  Another interpretation of this model is that it imposes a low-rank constraint on the parameters. As opposed to our approach, in the base case of using multiple pairwise models to perform multilingual translation, each model has \(P = P^{(enc)} + P^{(dec)}\) learnable parameters for its encoder and decoder. Given that the models are pairwise, for \(L\) languages, we have a total of \(L(L - 1)\) learnable parameter vectors of size \(P\). On the other hand, using our contextual parameter generator we have a total of \(L\) vectors of size \(M\) (one for each language), and a single matrix of size \(P \times M\). Then, the parameters of the encoder and the decoder, for a single language pair, are defined as a linear combination of the \(M\) columns of that matrix, as shown in the above equations. In our <a href="http://aclweb.org/anthology/D18-1039" target="_blank">EMNLP 2018 paper</a>, we consider and discuss more options for the parameter generator network, that allow for controllable parameter sharing.
</div>

<h4 class="title mt-4 p-0 text-left">Generalization</h4>

<div class="col mt-4 p-0">
  The contextual parameter generator is a generalization of previous approaches for multilingual NMT:
</div>

<ol class="mt-2">
  <li>
    <b>Pairwise:</b> \(g\) picks a different parameter set based on the language pair.
  </li>
  <li>
    <b>Universal:</b> \(g\) picks the same parameters for all languages.
  </li>
  <li>
    <b>Per-Language Encoder/Decoder:</b> \(g\) picks a different set of encoder/decoder parameters based on the languages.
  </li>
</ol>

<h4 class="title mt-4 p-0 text-left">Semi-Supervised and Zero-Shot Learning</h4>

<div class="col mt-4 p-0">
  The parameter generator also enables <i>semi-supervised learning</i>. Monolingual data can be used to train the shared encoder/decoder networks to translate a sentence from some language to itself (similar to the idea of <a href="https://en.wikipedia.org/wiki/Autoencoder" target="_blank">auto-encoders</a>). This is possible and can help learning because many of the learnable parameters are shared across languages.
</div>

<div class="col mt-2 p-0">
  Furthermore, <i>zero-shot translation</i>, where the model translates between language pairs for which it has seen no explicit training data, is also possible. This is because the same per-language parameters are used to translate to and from a given language, irrespective of the language at the other end. Therefore, as long as we train our model using some language pairs that involve a given language, it is possible to learn to translate in any direction involving that language.
</div>

<h4 class="title mt-4 p-0 text-left">Potential for Adaptation</h4>

<div class="col mt-4 p-0">
  Let us assume that we have trained a model using data for some set of languages, \(\ell_1, \ell_2, \dots, \ell_m\). If we obtain data for some new language \(\ell_n\), we do not have to retrain the whole model from scratch. In fact, we can fix the parameters that are shared across all languages and only learn the embedding for the new language (along with the relevant word embeddings if not using a shared vocabulary). Assuming that we had a sufficient number of languages in the beginning, this may allow us to obtain reasonable translation performance for the new language, with a minimal amount of training (due to the small number of parameters that need to be learned in this case --- to put this into perspective, in most of our experiments we used language embeddings of size 8).
</div>

<h4 class="title mt-4 p-0 text-left">Number of Parameters</h4>

<div class="col mt-4 p-0">
  For the base case of using multiple pairwise models to perform multilingual translation, each model has \(P + 2WV\) parameters, where \(P = P^{(enc)} + P^{(dec)}\), \(W\) is the word embedding size, and \(V\) is the vocabulary size per language (assumed to be the same across languages, without loss of generality). Given that the models are pairwise, for \(L\) languages, we have a total of \(L(L - 1)(P + 2WV)\)\(=\mathcal{O}(L^2P +2L^2WV)\) learnable parameters. For our approach, using the linear parameter generator network we have a total of \(\mathcal{O}(PM + LWV)\) learnable parameters. Note that the number of encoder/decoder parameters has no dependence on \(L\) now, meaning that our model can easily scale to a large number of languages.
</div>

<div class="col mt-2 p-0">
  For putting these numbers into perspective, in our experiments we had \(W = 512\), \(V = 20000\), \(L = 8\), \(M = 6\), and \(P\) in the order millions.
</div>

<h3 class="title mt-4 text-left p-0">Results</h3>

<div class="col mt-4 p-0">
  We present results from experiments on two datasets in our <a href="http://aclweb.org/anthology/D18-1039" target="_blank">paper</a>, but here we highlight some of the most interesting ones. In the following table we compare CPG to using the pairwise NMT models approach, as well as the universal one (using Google's multilingual NMT system). Below are results for the <a href="https://sites.google.com/site/iwsltevaluation2015/mt-track" target="_blank">IWSLT-15 dataset</a>, a commonly used small dataset in NMT community.
</div>

<table class="table col-12 col-sm-10 col-md-6 ml-auto mr-auto p-0">
  <thead>
    <tr>
      <th scope="col"></th>
      <th scope="col">Pairwise</th>
      <th scope="col">Universal</th>
      <th scope="col">CPG</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row" class="font-italic">En-Cs</th>
      <td>14.89</td>
      <td>15.92</td>
      <td class="font-weight-bold">17.22</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">Cs-En</th>
      <td>24.43</td>
      <td>25.25</td>
      <td class="font-weight-bold">27.37</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">En-De</th>
      <td>25.99</td>
      <td>25.92</td>
      <td class="font-weight-bold">26.77</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">De-En</th>
      <td>30.93</td>
      <td>29.60</td>
      <td class="font-weight-bold">31.77</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">En-Fr</th>
      <td>38.25</td>
      <td>34.40</td>
      <td class="font-weight-bold">38.32</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">Fr-En</th>
      <td>37.40</td>
      <td>35.14</td>
      <td class="font-weight-bold">37.89</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">En-Th</th>
      <td>23.62</td>
      <td>22.22</td>
      <td class="font-weight-bold">26.33</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">Th-En</th>
      <td>15.54</td>
      <td>14.03</td>
      <td class="font-weight-bold">26.77</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">En-Vi</th>
      <td>27.47</td>
      <td>25.54</td>
      <td class="font-weight-bold">29.03</td>
    </tr>
    <tr>
      <th scope="row" class="font-italic">Vi-En</th>
      <td>24.03</td>
      <td>23.19</td>
      <td class="font-weight-bold">26.38</td>
    </tr>
    <tr>
      <th scope="row" class="font-weight-bold">Mean</th>
      <td>26.26</td>
      <td>25.12</td>
      <td class="font-weight-bold">27.80</td>
    </tr>
  </tbody>
</table>

<div class="col mt-4 p-0">
  The numbers shown in this table represent <a href="https://en.wikipedia.org/wiki/BLEU" target="_blank">BLEU scores</a>, the most widely used metric for evaluating MT systems. They represent a measure of precision over \(n\)-grams. More specifically, in this instance they represent a measure of how frequently the MT prediction outputs 4 consecutive words that match 4 consecutive words in the reference translation.
</div>

<div class="col mt-2 p-0">
  We show similar results for the <a hreff="https://sites.google.com/site/iwsltevaluation2017/TED-tasks" target="_blank">IWSLT-17 dataset</a> (commonly used challenge dataset for multilingual NMT systems that includes a zero-shot setting) in our <a href="http://aclweb.org/anthology/D18-1039" target="_blank">paper</a>, where CPG outperforms both the pairwise approach and the universal approach (i.e., Google's system). Most interestingly, we computed the <a href="http://cosine%20distance" target="_blank">cosine distance</a> between all pairs of language embeddings learned by CPG:
</div>

<div class="container-fluid text-center mt-4 p-0">
  <img class="img-responsive col-12 col-sm-10 col-md-6 ml-auto mr-auto" src="{{ '/assets/img/projects/machine_translation/6_languages.png' | prepend: site.baseurl | prepend: site.url }}" alt="overview figure">
  <h6 class="font-italic text-center mt-2" style="color: #78909c;"><u>Figure 6:</u> <a href="http://cosine%20distance" target="_blank">Cosine distance</a> between all pairs of the language embeddings learned using our contextual parameter generator. Note that the cosine distance values can range between 0 and 2.</h6>
</div>

<div class="col mt-4 p-0">
  There are some interesting patterns that indicate that the learned language embeddings are reasonable. For example, we observe that German (<i>De</i>) and Dutch (<i>Nl</i>) are most similar for the IWSLT-17 dataset, with Italian (<i>It</i>) and Romanian (<i>Ro</i>) coming second. Furthermore, Romanian and German are the furthest apart for that dataset. It is very encouraging to see that these relationships agree with linguistic knowledge about these languages and the families they belong to. We see similar patterns in the IWSLT-15 results but we focus on IWSLT-17 here, because it is a larger, better quality dataset with more supervised language pairs. These results also uncover relationships between languages that may have been previously unknown. For example, perhaps surprisingly, French (<i>Fr</i>) and Vietnamese (<i>Vi</i>) appear to be significantly related for the IWSLT-15 dataset results. This is likely due to <a href="https://en.wikipedia.org/wiki/French_language_in_Vietnam" target="_blank">French influence in Vietnamese</a> due to the occupation of Vietnam by France during the 19th and 20th centuries.
</div>

<h3 class="title mt-4 p-0 text-left">Where should you look to learn more?</h3>

<div class="col mt-4 p-0">
  More details can be found in our <a href="http://aclweb.org/anthology/D18-1039" target="_blank">EMNLP 2018 paper</a>. We have also released an implementation of our approach and experiments as part of a new <a href="https://github.com/eaplatanios/symphony-mt" target="_blank">Scala framework for machine translation</a>. It is built on top of <a href="https://github.com/eaplatanios/tensorflow_scala" target="_blank">TensorFlow Scala</a> and follows a modular NMT design that supports various NMT models, including our baselines. It also contains data loading and preprocessing pipelines that support multiple datasets and languages, and is more efficient than other packages (e.g., <a href="https://github.com/tensorflow/nmt" target="_blank">tf-nmt</a>). Furthermore, the framework supports various vocabularies, among which we provide a new implementation for the byte-pair encoding (BPE) algorithm that is 2 to 3 orders of magnitude faster than the <a href="https://github.com/rsennrich/subword-nmt" target="_blank">released one</a>.
</div>

<div class="col mt-2 p-0">
  I also plan to release another blog post later on in 2019 with some follow-up work we have done on applying contextual parameter generation to other problems.
</div>

<h3 class="title mt-4 p-0 text-left">Acknowledgements</h3>

<div class="col mt-4 p-0">
  We would like to thank Otilia Stretcu, Abulhair Saparov, and Maruan Al-Shedivat for the useful feedback they provided in early versions of this paper. This research was supported in part by AFOSR under grant FA95501710218.
</div>

<h3 class="title mt-4 p-0 text-left">References</h3>

<div class="col mt-4 p-0 tiny" style="color: #78909c;">
  Emmanouil Antonios Platanios, Mrinmaya Sachan, Graham Neubig, and Tom Mitchell. 2018. <a href="http://aclweb.org/anthology/D18-1039" target="_blank"><i>Contextual Parameter Generation for Universal Neural Machine Translation</i></a>. In Conference on Empirical Methods in Natural Language Processing (EMNLP), Brussels, Belgium.
  <br/><br/>
  Orhan Firat, Kyunghyun Cho, and Yoshua Bengio. 2016a. <a href="http://www.aclweb.org/anthology/N16-1101" target="_blank"><i>Multi-Way, Multilingual Neural Machine Translation with a Shared Attention Mechanism</i></a>. In Proceedings of NAACL-HLT, pages 866–875.
  <br/><br/>
  Thanh-Le Ha, Jan Niehues, and Alexander Waibel. 2016. <a href="https://workshop2016.iwslt.org/downloads/IWSLT_2016_paper_5.pdf" target="_blank"><i>Toward Multilingual Neural Machine Translation with Universal Encoder and Decoder</i></a>. In Proceedings of the 13th International Workshop on Spoken Language Translation.
  <br/><br/>
  Melvin Johnson, Mike Schuster, Quoc V Le, Maxim Krikun, Yonghui Wu, Zhifeng Chen, Nikhil Thorat, Fernanda Viegas, Martin Wattenberg, Greg Corrado, et al. 2017. <a href="https://www.aclweb.org/anthology/Q/Q17/Q17-1024.pdf" target="_blank"><i>Google’s Multilingual Neural Machine Translation System: Enabling Zero-Shot Translation</i></a>. In Transactions of the Association for Computational Linguistics, volume 5, pages 339–351.
  <br/><br/>
  Minh-Thang Luong, Quoc V Le, Ilya Sutskever, Oriol Vinyals, and Lukasz Kaiser 2016. <a href="https://arxiv.org/abs/1511.06114" target="_blank"><i>Multi-task Sequence to Sequence Learning</i></a>. In International Conference on Learning Representations.
</div>

{% endcomment %}
