---
layout: project
title: Current Mode AdEx Neuron
nav: projects
importance: 120
description: ultra low power neuron implementation for large scale neuromorphic cores
img: /assets/img/neuron.png
---

<b>Dynamic Translinear Principle(DTP)</b> has proven to be an effective technique to map neuronal membrane equations following sophisticated computational models on energy-efficient subthreshold MOSFETs. The technique allows to closely mimic brain's analog computational dynamics using minimum transistor resourses. DTP requires biological neuron models to be converted into dimensionless form. The dimensionless models could then easily be realized and synthesized onto <b>Log-Domain Circuits</b>(a class of circuits based on subthreshold MOSFETs following an exponential I-V characteristics), where a dimensionless variable \\(x\\) is represented by the ratio of two currents.

$$x = \frac{I_1}{I_2}$$

For any circuit, mean power consumed is the simple multiplication of the supply voltage with the average current drawn from it over a period of time (\\(P_{avg} = VDD*I_{avg}\\)). Log-Domain circuits allow aggressive down-scaling of the supply voltage because of their inherent ability to provide high dynamic range. This benefit has been explored in a great extent in many low-power biomedical/implantable circuits. This research project particularly focuses on reducing the current that the circuit consumes in order to lower the power consumption. Since the values of interest in the circuit realization are <b>ratios of currents, and not absolute currents</b>, the values of currents \\(I_1\\) and \\(I_2\\) are reduced while maintaining the ratio of interest. This allows aggressive down-scaling of power(potentially by orders of magnitude) by exploiting both voltage and current-domain fronts of log-domain circuits, and brings us closer to achieving the brain's energy density.

This research explores many techniques including <b>Reverse Body Biasing</b>, <b>Source Voltage Shifting</b>, and <b> Bulk-Driven Circuits</b> as methods to drastically reduce absolute currents consumed by log-domain neurons. These techniques are used in very specific log-domain topologies to leverage their benefits, while suppressing any avoidable second-order effects. The widely popular biological-time, two-dimensional <b>Adaptive Exponential(AdEx) Integrate and Fire Neuron</b> model is implemented using the above mentioned techniques to save more than 10 times power than the present state-of-the-art AdEx implementations.

<div class="alert alert-danger" role="alert">
  <b>NOTE:</b> This page will soon be updated with all the findings and the actual paper link.
</div>



{%comment%}

Designing ultra-energy-efficient artificial neurons is rudimentary to create brain-like intelligent and highly integrated computing systems. [A Neuromorph's Prospectus] shows that brain achieves it's energy density by doing its computation in the analog-domain, and communication in the digital domain. Analog computation facilitates erroneous low power operation by using extremely low currents and voltages to process the synaptic inputs. Log-domain   


Overture: the challenges of neuron architectures. want to connect neurons to 1000s of others. same time want to go to as low currents as possible. leakage stops us from going to lower currents. further more, log-domain works on low currents. reducing the currents to extremely low values using leakage reduction techniques.

Understanding the complicated and large scale synaptic connections in the critical areas of the brain, and mapping them onto energy-efficient brain-like circuits is the next challenge in brain inspired computing. Achieving extremely low energy density computations could make our relentless pursuit to powerful computing systems go on. It could also be the key to some of the technological advancements which could arguably dictate humanity's future for the next century. We have just started to harness the power of brain on circuits, and there are already cornerstone strides happening in the fields of Autonomous Robotic Systems, Cybersecurity, Financial Technology, amd Mobile Computing. On our way to build intelligent computer, which could pass the Turing test, and further down the road, a silicon consciousness, conflated with the wireless-advancements(6G), could create a cyborg reality, with systems controlled wirelessly by super-intelligent computers. All of this starts by achieving efficient on-chip neurons. Many    

The key to energy efficient scalability, and training of the circuit parameters lies in the neuron model mimicking the neuronal activity. Low-dimensional neuronal models have proven to provide enough biological accuracy, while allowing pragmatic tuability of parameters. Parallelly, research on log-domain circuits has allowed the design of low-voltage, high dynamic range circuits, also allowing the implementation of complex mathematical functions. Many log domain implementations of various computational models have been reported so far. Neurogrid uses analog computation, and digital communication, just like the brain in order to achieve brain's energy efficiency.

Using log-domain concept, the dimensionless computational models could be mapped on the circuits, and the currents in the specific branches represent variables corresponding to the computational models. We articulate methods to reduce the current consumption of log domain topologies, and implementing certain neuron models.    


This library is a Scala API for [https://www.tensorflow.org](https://www.tensorflow.org). It attempts to provide most of
the functionality provided by the official Python API, while at the same type being strongly-typed and adding some new
features. It is a work in progress and a project I started working on for my personal research purposes. Much of the API
should be relatively stable by now, but things are still likely to change. That is why there is no official release of
this library yet.

Please refer to the main website for documentation and tutorials. Here
are a few useful links:

  - [Installation](https://eaplatanios.github.io/tensorflow_scala/installation.html)
  - [Getting Started Guide](https://eaplatanios.github.io/tensorflow_scala/getting_started.html)
  - [Library Architecture](https://eaplatanios.github.io/tensorflow_scala/architecture.html)
  - [Contributing](https://eaplatanios.github.io/tensorflow_scala/contributing.html)

## Main Features

- Easy manipulation of tensors and computations involving tensors (similar to NumPy in Python):

  ```scala
  val t1 = Tensor( 1.2, 4.5)
  val t2 = Tensor(-0.2, 1.1)
  t1 + t2 == Tensor(1.0, 5.6)
  ```

- High-level API for creating, training, and using neural networks. For example, the following code shows how simple it
  is to train a multi-layer perceptron for MNIST using TensorFlow for Scala. Here we omit a lot of very powerful
  features such as summary and checkpoint savers, for simplicity, but these are also very simple to use.

  ```scala
  import org.platanios.tensorflow.api._
  import org.platanios.tensorflow.api.tf.learn._
  import org.platanios.tensorflow.data.loaders.MNISTLoader

  // Load and batch data using pre-fetching.
  val dataSet = MNISTLoader.load(Paths.get("/tmp"))
  val trainImages = DatasetFromSlices(dataSet.trainImages)
  val trainLabels = DatasetFromSlices(dataSet.trainLabels)
  val trainData =
    trainImages.zip(trainLabels)
        .repeat()
        .shuffle(10000)
        .batch(256)
        .prefetch(10)

  // Create the MLP model.
  val input = Input(UINT8, Shape(-1, 28, 28))
  val trainInput = Input(UINT8, Shape(-1))
  val layer = Flatten() >> Cast(FLOAT32) >>
      Linear(128, name = "Layer_0") >> ReLU(0.1f) >>
      Linear(64, name = "Layer_1") >> ReLU(0.1f) >>
      Linear(32, name = "Layer_2") >> ReLU(0.1f) >>
      Linear(10, name = "OutputLayer")
  val trainingInputLayer = Cast(INT64)
  val loss = SparseSoftmaxCrossEntropy() >> Mean()
  val optimizer = GradientDescent(1e-6)
  val model = Model(input, layer, trainInput, trainingInputLayer, loss, optimizer)

  // Create an estimator and train the model.
  val estimator = Estimator(model)
  estimator.train(trainData, StopCriteria(maxSteps = Some(1000000)))
  ```

  And by changing a few lines to the following code, you can get checkpoint capability, summaries, and seamless
  integration with TensorBoard:

  ```scala
  loss = loss >> tf.learn.ScalarSummary("Loss")                  // Collect loss summaries for plotting
  val summariesDir = Paths.get("/tmp/summaries")                 // Directory in which to save summaries and checkpoints
  val estimator = Estimator(model, Configuration(Some(summariesDir)))
  estimator.train(
    trainData, StopCriteria(maxSteps = Some(1000000)),
    Seq(
      SummarySaverHook(summariesDir, StepHookTrigger(100)),      // Save summaries every 1000 steps
      CheckpointSaverHook(summariesDir, StepHookTrigger(1000))), // Save checkpoint every 1000 steps
    tensorBoardConfig = TensorBoardConfig(summariesDir))         // Launch TensorBoard server in the background
  ```

  If you now browse to `https://127.0.0.1:6006` while training, you can see the training progress:

  <div class="col">
    <img src="{{ '/assets/img/tensorboard_mnist_example_plot.png' | prepend: site.baseurl | prepend: site.url }}" alt="tensorboard_mnist_example_plot" width="100%">
  </div>

- Low-level graph construction API, similar to that of the Python API, but strongly typed wherever possible:

  ```scala
  import org.platanios.tensorflow.api._

  val inputs = tf.placeholder(FLOAT32, Shape(-1, 10))
  val outputs = tf.placeholder(FLOAT32, Shape(-1, 10))
  val predictions = tf.createWith(nameScope = "Linear") {
    val weights = tf.variable("weights", FLOAT32, Shape(10, 1), tf.zerosInitializer)
    tf.matmul(inputs, weights)
  }
  val loss = tf.sum(tf.square(predictions - outputs))
  val optimizer = tf.train.AdaGrad(1.0)
  val trainOp = optimizer.minimize(loss)
  ```

- Numpy-like indexing/slicing for tensors. For example:

  ```scala
  tensor(2 :: 5, ---, 1) // is equivalent to numpy's 'tensor[2:5, ..., 1]'
  ```

- Efficient interaction with the native library that avoids unnecessary copying of data. All tensors are created and
  managed by the native TensorFlow library. When they are passed to the Scala API (e.g., fetched from a TensorFlow
  session), we use a combination of weak references and a disposing thread running in the background. Please refer to
  [`Disposer.scala`](https://github.com/eaplatanios/tensorflow_scala/blob/master/modules/api/src/main/scala/org/platanios/tensorflow/api/utilities/Disposer.scala),
  for the implementation.

## Funding

Funding for the development of this library has been generously provided by the following sponsors:

<div class="col table-responsive">
<table class="table table-sm table-borderless">
  <tr>
    <td class="p-1 text-center align-middle" style="width: 30%;" >
      <img class="img-responsive" style="width: 80%;" src="{{ '/assets/img/cmu_logo.svg' | prepend: site.baseurl | prepend: site.url }}" alt="cmu_logo">
    </td>
    <td class="p-1 text-center align-middle" style="width: 30%;" >
      <img class="img-responsive" style="width: 80%;" src="{{ '/assets/img/nsf_logo.svg' | prepend: site.baseurl | prepend: site.url }}" alt="cmu_logo">
    </td>
    <td class="p-1 text-center align-middle" style="width: 30%;" >
      <img class="img-responsive" style="width: 80%;" src="{{ '/assets/img/afosr_logo.gif' | prepend: site.baseurl | prepend: site.url }}" alt="cmu_logo">
    </td>
  </tr>
  <tr>
    <td class="p-1 font-weight-light text-center"><b>CMU Presidential Fellowship</b> <br/>awarded to <br/>Emmanouil Antonios Platanios</td>
    <td class="p-1 font-weight-light text-center"><b>National Science Foundation</b> <br/>Grant #: IIS1250956</td>
    <td class="p-1 font-weight-light text-center"><b>Air Force Office of Scientific Research</b> <br/>Grant #: FA95501710218</td>
  </tr>
</table>
</div>

{%endcomment%}
