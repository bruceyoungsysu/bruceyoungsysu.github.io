---
title: Picture Style Transfer with CNN
categories:
 - CNN
 - Style Transfer
tags: CNN
image: https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/style_transfer/Deadpool_sn.png
---

Any tranformaton involving frequency domain operations is suitable for image processing. An image can be treated as a two-dimensional signal which can be applied many image processing techniques. High dimensional Fourier Transformation, Convolution and even Wavelet Transformation are suitable to extract certain features from a 2D signal with possibilities of various filters.

# Why CNN is Different

Basically we can consider two ways of interpreting the content of an image: one way is to treat the image in time domain, thus we can expand the image by pixel forming them to 1D vector. 

![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/classifier_on_CNN/one_dimensional.png) 

Thus the one dimensional vector can be used in any classifier, not only Neural Network, but also SVM and random forest. 

Or we can treat it as a two dimensional signal as we mentioned above. Then train the train the filters on its extracted signals. Thus we actually updating this network on frequency domain.

![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/classifier_on_CNN/conv.png)

(All figures from: http://cs231n.stanford.edu/slides/2018/cs231n_2018_lecture05.pdf)

# How to Build a CNN

Here we will build a CNN for digit recognizer, for detail of the task please refer this kaggle page https://www.kaggle.com/c/digit-recognizer. The code refers to the convnet on Minist dataset example at Stanford https://github.com/chiphuyen/stanford-tensorflow-tutorials/blob/master/examples/07_convnet_mnist.py.

There are several certain steps need to be configured before running the network:

- Define the output size. Meanwhile the batch size is also needs to be well defined. We can use `tf.data.Iterator.from_structure` to make a iterator generating the batch size of input data each time.

  ```python
          with tf.name_scope('data'):
              train, test = parse_data(self.train_path, self.test_path)
              # load data with tf.data.Dataset
              train_data = tf.data.Dataset.from_tensor_slices(train)
              test_data = tf.data.Dataset.from_tensor_slices(test)
              # define the data batches
              train_data = train_data.shuffle(10000)
              train_data = train_data.batch(4000)
              test_data = test_data.batch(4000)
              # make a batch data interator
              iterator = tf.data.Iterator.from_structure(train_data.output_types, train_data.output_shapes)
              img, label = iterator.get_next()
              # reshape the input and target
              self.label = tf.one_hot(indices=label, depth=10, on_value=1, off_value=0)
              self.img = tf.reshape(img, [-1, 28, 28, 1])
              # make initializers for iterator
              self.train_init = iterator.make_initializer(train_data)
              self.test_init = iterator.make_initializer(test_data)
  ```

- Define the structure of CNN, i.e. how many convolutional layers and how many pooling layers, and their types. Then define the output form by setting an output layer.

  ```python
      def inference(self):
          # define conv and pooling layers
          conv1 = conv_relu(self.img, filters=32, k_size= 5, stride=1, padding='SAME',scope_name= 'conv1')
          pool1 = maxpool(conv1, ksize=2, stride=1, padding='SAME', scope_name='pool1')
          conv2 = conv_relu(pool1, filters=64, k_size=5, stride=1, padding='SAME', scope_name='conv2')
          pool2 = maxpool(conv2, ksize=2, stride=1, padding='SAME', scope_name='pool2')
          # reshape the pooling layer final output
          feature_shape = pool2.shape[1]*pool2.shape[2]*pool2.shape[3]
          pool2 = tf.reshape(pool2, [-1, feature_shape])
          # feed into output layer, dropout to prevent overfitting
          fc1 = fully_connected(pool2, 1024, scope_name='fc1')
          dropout = tf.nn.dropout(tf.nn.relu(fc1), self.keep_prob, name='relu_dropout')
          self.logits = fully_connected(dropout, 10, scope_name='logits')
  ```

- Define the loss function and optimizer

  ```python
      def loss(self):
          with tf.name_scope('loss'):
              entropy = tf.nn.softmax_cross_entropy_with_logits(labels=self.label, logits=self.logits)
              self.loss = tf.reduce_mean(entropy, name='loss')
  
      def optimize(self):
          self.optimizor = tf.train.AdamOptimizer(self.lr).minimize(self.loss)
  ```

  The loss function is the entropy between the labels and outputs, if we are more interested in long distance prediction, we could make the sequence shift between target and prediction bigger. 

There are more details about how convolution and pooling are done mathematically, this can be referred from the nice note from Stanford:http://cs231n.github.io/convolutional-networks/.