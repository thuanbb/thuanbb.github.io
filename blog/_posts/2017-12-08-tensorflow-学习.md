---
title: Tensorflow 学习
updated: 2017-12-08 14:00
category: 
tag: [deep learning]
---

# Document


### Tensorflow building blocks

1. Data tensors `node = tf.constant(3.0, dtype=tf.float32)`
3. Variables `a = tf.placeholder(tf.float32)`
2. Dependent variables `adder_node = tf.add(a, b)`
4. Executive session `sess = tf.Session()`, `sess.run(adder_node, {a: 3, b: 4.5})`
5. Trainable parameters `W = tf.Variable([.3], dtype=tf.float32)`
6. Parameter initialization `init = tf.global_variables_initializer()`, `sess.run(init)`
7. Parameter assignment `fixW = tf.assign(W, [-1.])`, `sess.run(fixW)`
8. Optimizer `optimizer = tf.train.GradientDescentOptimizer(0.01)`
9. Trainer=optimizer+loss `train = optimizer.minimize(loss)`
10. High-level training toolkit `tf.estimator`
11. Saving variables: `saver = tf.train.Saver()`, `saver.save(sess, "/tmp/model.ckpt")`
12. Restore variables: `saver.restore(sess, "/tmp/model.ckpt")`

### Using GPUs

Device mapping management:
1. `with tf.device('/device/GPU:0'):` Specify computing devices for operators.
2. `sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))
` Output device mapping into log.

GPU memory management:
3. `config = tf.ConfigProto() config.gpu_options.allow_growth = True `
allocate only as much GPU memory based on runtime allocations
4. `config.gpu_options.per_process_gpu_memory_fraction = 0.4` bound the amount of GPU memory available to the TensorFlow process

### Network layers

```python
tf.layers.conv2d()	# conv2d
tf.layers.conv3d()	# conv3d
tf.layers.conv2d_transpose()	# deconv2d
tf.layers.conv3d_transpose()	# deconv3d
tf.layers.max_pooling2d()	# max pooling
tf.layers.average_pooling()	# avg pooling
tf.layers.dense(input, units)	# fc, output dimension = units
tf.layers.batch_normalization()	# bn
tf.layers.dropout(inputs, rate=0.5, training)	# dropout, 40% of elements are dropped out only in training mode

# Activations
tf.nn.relu()	# relu
tf.nn.dropout()
tf.sigmoid()	# sigmoid
tf.tanh()	# tanh

tf.nn.local_response_normalization()	# lrn
tf.nn.softmax()	# softmax
tf.nn.l2_normalization()	# l2norm

tf.concat()	# concat
tf.add_n()	# add
tf.squeeze()	# Removes dimensions of size 1 from the shape of a tensor
tf.maximum(x, y, name)		# element-wise maximum
tf.tanh(x, name)
tf.reshape(tensor, shape, name)
tf.slice(tensor, begin, size, name)	# get slice of tensor
tf.gather(slices, indices, axis=0, name)	# put together slices but follow the order of indices
tf.stack(tensor, axis, name)	# stack a list of rank-R tensors into a rank-(R+1) one
tf.unstack(tensor, num, axis, name)	# chipping rank-(R+1) tensor along axis into a amount of num rank-R tensors
tf.argmax(tensor, axis)
```

### Tesorflow变量名
`tf.Variable()`和`tf.get_variable()`是新建变量的两种方式。
`tf.Variable()`每次都会产生新的变量，当变量名重复时会引入**别名机制**自行处理。
`tf.get_variable()`遇到重名的变量且变量名没有被设置成共享时会直接报错。

`tf.name_scope()`和`tf.variable_scope()`
`tf.name_scope()`用于对图里面的各种operator进行层次化的管理，类似C++中的`namespce`。
`tf.variable_scope()`除了`tf.name_scope()`的功能之外，还允许在一个variable_scope下共享变量：
```python
with tf.variable_scope('my_scope') as scope:
	var1 = tf.get_variable(name='var1', shape=[1], dtype=tf.float32)
	scope.reuse_variables()
	var1_reuse = tf.get_variable(name='var1')
```

### Estimators
* Define Estimator: `tf.estimator` are essentially predefined models. The examples are shown below:

```python
# feature columns specify data type of input
feature_columns = [tf.feature_column.numeric_column("x", shape=[4])]

# Build 3 layer DNN with 10, 20, 10 units respectively.
estimator = tf.estimator.DNNClassifier(feature_columns=feature_columns,
                                        hidden_units=[10, 20, 10],
                                        n_classes=3,
                                        model_dir="/checkpoints/save/path")
# Customized Estimator
def model_fn(features, labels, mode, params):
	optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.001)
	train_op = optimizer.minimize(loss=loss, global_step=tf.train.get_global_step())
	# global_step is a counter of training steps
	# define loss function
	return tf.estimator.EstimatorSpec(mode=mode, loss=loss, train_op=train_op)
	# return EstimatorSpec that fully defines the model to be run by Estimator

estimator = tf.estimator.Estimator(
    	model_fn=model_fn, 
	model_dir="/checkpoints/save/path",
	l1_regularization_strength=1.0,
        l2_regularization_strength=1.0)
	# L1 regularization tends to make model weights stay at zero
	# L2 regularization also tries to make model weights closer to zero but not necessarily zero
```

* Define input function: The input builder function constructs input data in form of `tf.Tensor` and `tf.SparseTensor` (mostly for categorical data).

```python
# Define the training inputs
train_input_fn = tf.estimator.inputs.numpy_input_fn(
    x={"x": np.array(training_set.data)},
    y=np.array(training_set.target),
    num_epochs=None,
    shuffle=True)

# Define customized input function
def dataset_input_fn():
  filenames = ["/var/data/file1.tfrecord", "/var/data/file2.tfrecord"]
  dataset = tf.data.TFRecordDataset(filenames)

  # Use `tf.parse_single_example()` to extract data from a `tf.Example`
  # protocol buffer, and perform any additional per-record preprocessing.
  def parser(record):
    keys_to_features = {
        "image_data": tf.FixedLenFeature((), tf.string, default_value=""),
        "date_time": tf.FixedLenFeature((), tf.int64, default_value=""),
        "label": tf.FixedLenFeature((), tf.int64,
                                    default_value=tf.zeros([], dtype=tf.int64)),
    }
    parsed = tf.parse_single_example(record, keys_to_features)

    # Perform additional preprocessing on the parsed data.
    image = tf.decode_jpeg(parsed["image_data"])
    image = tf.reshape(image, [299, 299, 1])
    label = tf.cast(parsed["label"], tf.int32)

    return {"image_data": image, "date_time": parsed["date_time"]}, label

  # Use `Dataset.map()` to build a pair of a feature dictionary and a label
  # tensor for each example.
  dataset = dataset.map(parser)
  dataset = dataset.shuffle(buffer_size=10000)
  dataset = dataset.batch(32)
  dataset = dataset.repeat(num_epochs)
  iterator = dataset.make_one_shot_iterator()

  # `features` is a dictionary in which each value is a batch of values for
  # that feature; `labels` is a batch of labels.
  features, labels = iterator.get_next()
  return features, labels
# feature_cols: a dictionary containing key/value pairs that map feature column names to Tensors
# labels: A tensor containing label values
```

* Train, Evaluate or Predict

```python
# Train model.
classifier.train(input_fn=train_input_fn, steps=2000)
# Evaluate accuracy.
accuracy_score = classifier.evaluate(input_fn=test_input_fn)["accuracy"]
# Predict new samples.
predictions = list(classifier.predict(input_fn=predict_input_fn))
```

### Graph visualization
Inherently, a graph is built after we define tensors or variables that flow through operation nodes. 
Then the graph can be save into log dir by
```
with tf.Session() as sess:
    writer = tf.summary.FileWriter(output_folder, sess.graph)
    sess.run(graph_fn)
    writer.close()
```
To visualize the graph, just run command `tensorboard --logdir /your/dir/` if tensorflow is installed.

### Debug

* Debug with session: just warp session with a debugger wrapper, then debug in CLI.

```python
from tensorflow.python import debug as tf_debug
sess = tf_debug.LocalCLIDebugWrapperSession(sess)
```

* Debug with estimator: Estimator mannages sessions internally.

```python
# Create a LocalCLIDebugHook and use it as a monitor when calling fit().
hooks = None
debug_hook = tf_debug.LocalCLIDebugHook(ui_type=FLAGS.ui_type,
                                        dump_root=FLAGS.dump_root)
hooks = [debug_hook]

if not FLAGS.use_experiment:
	# Fit model.
	classifier.fit(x=training_set.data,
                   y=training_set.target,
                   steps=FLAGS.train_steps,
                   monitors=hooks)

	# Evaluate accuracy.
	accuracy_score = classifier.evaluate(x=test_set.data,
				y=test_set.target,
				hooks=hooks)["accuracy"]
```

### Visiualize embedding

> An embedding is a mapping from discrete **objects**, such as words, articles, images, to **vectors** of real numbers. In this way, the similarity in vector space can be used as a flexible and robust measure object similarity, e.g., `blue:  (red, 47.6°), (yellow, 51.9°), (purple, 52.4°)`.

[https://www.tensorflow.org/programmers_guide/embedding](https://www.tensorflow.org/programmers_guide/embedding)
[https://www.tensorflow.org/versions/r0.12/how_tos/embedding_viz/](https://www.tensorflow.org/versions/r0.12/how_tos/embedding_viz/)
