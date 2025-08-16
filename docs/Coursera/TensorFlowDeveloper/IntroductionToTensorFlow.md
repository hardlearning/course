# Introduction to TensorFlow for Artificial Intelligence, Machine Learning, and Deep Learning

## Week1: A New Programming Paradigm

```python
model = keras.Sequential([keras.layers.Dense(units=1, input_shape=[1])])
model.compile(optimizer='sgd', loss='mean_squared_error')

xs = np.array([-0.1, 0.0, 1.0, 2.0, 3.0, 4.0])
ys = np.array([-0.3, -1.0, 1.0, 3.0, 5.0, 7.0])

model.fit(xs, ys, epochs=500)

print(model.predict([10.0])
```

## Week2: Introduction to Computer Vision

### An Introduction to computer vision

**Fashion MNIST**:

- 70k Images
- 10 Categories
- Images are 28 by 28 pixels

Each pixel can be represented in values from zero to 255 and so it's only one byte per pixel. With 28 by 28 pixels in an image, only 784 byres are needed to store the entire image.

### Writing code to load taining data

In the Fashion-MNIST data set, 60,000 of the 70,000 images are used to train the network, and then 10,000 images can be used to test just how good or how bad it is performing.

### The stucture of Fahion MNIST data

Using a number label is a first step in avoiding bias -- instead of labelling it with words in a specific language and excluding people who don't speak that language.

### Coding a Computer Vision Neural Network

The last layer has 10 neurons in it because we have ten classes of clothing in the datasets.

The first layer is a flatten layer with the input shaping 28 by 28. Because our images are 28 by 28, so we're specifying that this is the shape that we should expect the data to be in.

Flattern takes the 28 by 28 square and turns it into a simple linear array.

The middle layer simetimes also called a hidden layer. This is a 128 neurons in it.

The function was y equals w1 times x1, plus w2 times x2, plus w3 times x3, all the way up to a w128 times x128.

https://youtu.be/fXOsFF95ifk

### Walk through a Notebook for computer vision

Neural networks work better with normalized data. So let's change it to between zero and one simply by dividing every value by 255.

We compile the model to finding the loss function and the optimizer, and the goal of these is to make a guess as to what the relationship is between the input data and the output data, measure how well or how badly it did using the lossing function, use the optimizer to generate a new guess and repeat.

```python
import tensorflow as tf
from tensorflow import keras

mnist=tf.keras.datasets.fashion_mnist
(train_images, train_labels), (test_images, test_labels)=mnist.load_data()
train_images = train_images / 255.0
test_images = test_images / 255.0

model=keras.Sequential([
    keras.layers.Flatten(input_shape=(28, 28)),
    keras.layers.Dense(128, activation=tf.nn.relu),
    keras.layers.Dense(10, activation=tf.nn.softmax)
])
model.compile(optimizer=tf.optimizers.Adam(), loss='sparse_categorical_crossentropy')
model.fit(train_images, train_labels, epochs=5)

model.evaluate(test_images, test_labels)
```

### Using Callbacks to control training

The training loop does support callbacks. So in every epoch, you can callback to a code function, having checked the metrics. If they're what you want to say, then you can cancel the training at that point.

Callbacks is implemented as a separate class, but that can be in-line with your other code. It doesn't need to be in a separate file. In it, we'll implement the on_epoch_end function, which gets called by the callback whenever the epoch ends.It also sends a logs object which contains lots of great information about the current state of training. For example, the current loss is available in the logs, so we can query it for certain amount. For example, here I'm checking if the loss is less than 0.4 and canceling the training itself.

```python
class MyCallback(tf.keras.callbacks.Callback):
    def on_epoch_end(self,epoch,logs={}):
        if (logs.get('loss')<0.4):
            print("\nLoss is low so cancelling training!")
            self.model.stop_training = True

callbacks = MyCallback()
model.fit(train_images, train_labels, epochs=5,callbacks=[callbacks])
```

## Week3: Enhancing Vision with Convolutional Neural Networks

### What are convolutions and pooling?

If you've ever done any kind of image processing , it usually involves having a filter and passing that filter over the image in order to change the underlying image.

![convolution](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/convolution.jpg)

If our filter is three by three, then we can  take a look at the immediate neighbor, so that you have a corresponding three by three grid. Then to get the new value for the pixel, we simply multiply each neighbor by the corresponding value in the filter.

The idea here is that some convolutions will change the image in such a way that certain features in the image get emphasized.

Pooling is a way of compressing an image.


![pooling](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/pooling.jpg)

A quick and easy way to do this , is to go over the image of four pixels at a time, i.e, the current pixel and its neighbors underneath and to the right of it. Of these four, pick the biggest value and keep just that.

This will preserve the features that were highlighted by the convolution, while simultaneously quartering the size of the image.

### Implementing convolutional layers

Here we're specifying the first convolution. We're asking keras to generate 64 filters for us. These filters are 3 by 3, their activation is relu, which means the negative values will be thrown away, and finally the input shape is as before, the 28 by 28. That extra 1 just means that we are tallying using a single byte for color depth. As we saw before our image is our gray scale, so we just use one byte.

https://bit.ly/2UGa7uH

### Implementing pooling layers

It's a two-by-two pool, so for every four pixels, the biggest one will survive.

The model.summary method allows you to inspect the layers of the model, and see the journey of the image through the convolutions.

After all, isn't the data 28 by 28, so y is the output, 26 by 26. The key to this is remembering that the filter is a three by three filter. Consider what happens when you start scanning through an image starting on the top left.

![Implementing-pooling-layers-1](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Implementing-pooling-layers-1.png)

![Implementing-pooling-layers-2](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Implementing-pooling-layers-2.png)

So, for example with this image of the dog on the right, you can see zoomed into the pixels at its top left corner. You can't calculate the filter for the pixel in the top left, because it doesn't have any neighbors above it or to its left. In a similar fashion, the next pixel to the right won't work either because it doesn't have any neighbors above it. So, logically, the first pixel that you can do calculations on is this one, because this one of course has all eight neighbors that a three by three filter needs. This means that you can't use a one pixel margin all around the image, so the output of the convolution will be two pixels smaller on x, and two pixels smaller on y.

If your filter is five-by-five for similar reasons, your output will be four smaller on x, and four smaller on y.

Next is the first of the max-pooling layers. Now, remember we specified it to be two-by-two, thus turning four pixels into one, and having our x and y. So, now our output gets reduced from 26 by 26, to 13 by 13. The convolutions will then operate on that, and of course, we lose the one pixel margin as before, so we're down to 11 by 11, add another two-by-two max-pooling to have this rounding down, and went down, down to five-by-five images. 

It's not just one compress five-by-five image instead of the original 28 by 28, there are a number of convolutions per image that we specified, in this case 64. So, there are 64 new images of five-by-five that had been fed in. Flatten that out and you have 25 pixels times 64, which is 1600. So, you can see that the new flattened layer has 1,600 elements in it, as opposed to the 784 that you had previously. 

```
 Layer (type)                Output Shape              Param #   
=================================================================
 conv2d (Conv2D)             (None, 26, 26, 64)        640       
                                                                 
 max_pooling2d (MaxPooling2D  (None, 13, 13, 64)       0         
 )                                                               
                                                                 
 conv2d_1 (Conv2D)           (None, 11, 11, 64)        36928     
                                                                 
 max_pooling2d_1 (MaxPooling  (None, 5, 5, 64)         0         
 2D)                                                             
                                                                 
 flatten_2 (Flatten)         (None, 1600)              0         
                                                                 
 dense_4 (Dense)             (None, 128)               204928    
                                                                 
 dense_5 (Dense)             (None, 10)                1290      
                                                                 
=================================================================
```

### Improving the Fashion classifier with convolutions

We're going to do two convolutional layers each with 64 convolution, and each followed by a max pooling layer. You can see that we defined our convolutions to be three-by-three and our pools to be two-by-two.

For every image, 64 convolutions are being tried, and then the image is compressed and then another 64 convolutions, and then it's compressed again, and then it's passed through the DNN, and that's for 60,000 images that this is happening on each epoch. So it might take a few minutes instead of a few seconds.

The Keras API gives us each convolution and each pooling and each dense, etc. as a layer. So with the layers API, I can take a look at each layer's outputs, so I'll create a list of each layer's output. I can then treat each item in the layer as an individual activation model if I want to see the results of just that layer. Now, by looping through the layers, I can display the journey of the image through the first convolution and then the first pooling and then the second convolution and then the second pooling. Note how the size of the image is changing by looking at the axes. 

If I set the convolution number to one, we can see that it almost immediately detects the laces area as a common feature between the shoes.

![Improving-the-Fashion-classifier-1](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-1.png)

If I change the third image to be one, which looks like a handbag, you'll see that it also has a bright line near the bottom that could look like the sole of the shoes, but by the time it gets through the convolutions, that's lost, and that area for the laces doesn't even show up at all. So this convolution definitely helps me separate issue from a handbag.

![Improving-the-Fashion-classifier-2](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-2.png)

If I said it's a two, it appears to be trousers, but the feature that detected something that the shoes had in common fails again. 

![Improving-the-Fashion-classifier-3](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-3.png)

If I changed my third image back to that for shoe, but I tried a different convolution number, you'll see that for convolution two, it didn't really find any common features. 

![Improving-the-Fashion-classifier-4](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-4.png)

To see commonality in a different image, try images two, three, and five. These all appear to be trousers. 

![Improving-the-Fashion-classifier-5](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-5.png)

Convolutions two and four seem to detect this vertical feature as something they all have in common. 

![Improving-the-Fashion-classifier-6](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-6.png)

If I again go to the list and find three labels that are the same, in this case six, I can see what they signify. When I run it, I can see that they appear to be shirts. 

![Improving-the-Fashion-classifier-7](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-7.png)

Convolution four doesn't do a whole lot, so let's try five. We can kind of see that the color appears to light up in this case. 

![Improving-the-Fashion-classifier-8](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-8.png)

Let's try convolution one. 

![Improving-the-Fashion-classifier-9](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Improving-the-Fashion-classifier-9.png)

```python
model = tf.keras.models.Sequential([
    tf.keras.layers.Conv2D(64, (3,3), activation='relu',input_shape=(28, 28, 1)), 
    tf.keras.layers.MaxPooling2D(2, 2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'), 
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Flatten(), 
    tf.keras.layers.Dense(128, activation='relu'), 
    tf.keras.layers.Dense(10, activation='softmax')
])

model.summary()
```

### Walking through convolutions

Here, we can see the ascent image from SciPy. 

![Walking-through-convolutions-1](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-1.png)

So we can see here that it's a 512 by 512 image. So now, let's create a convolution as a three by three array. We'll load it with values that are pretty good for detecting sharp edges first. 

Here's where we'll create the convolution. We iterate over the image, leaving a one pixel margin. You'll see that the loop starts at one and not zero, and it ends at size x minus one and size y minus one. In the loop, it will then calculate the convolution value by looking at the pixel and its neighbors, and then by multiplying them out by the values determined by the filter, before finally summing it all up. 

We can see that only certain features made it through the filter.

![Walking-through-convolutions-2](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-2.png)

I've provided a couple more filters, so let's try them. This first one is really great at spotting vertical lines. So when I run it, and plot the results, we can see that the vertical lines in the image made it through. It's really cool because they're not just straight up and down, they are vertical in perspective within the perspective of the image itself.

![Walking-through-convolutions-3](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-3.png)

Similarly, this filter works well for horizontal lines. So when I run it, and then plot the results, we can see that a lot of the horizontal lines made it through.

![Walking-through-convolutions-4](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-4.png)

Now, let's take a look at pooling, and in this case, Max pooling, which takes pixels in chunks of four and only passes through the biggest value. I run the code and then render the output. 

![Walking-through-convolutions-5](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-5.png)

We can see that the features of the image are maintained, but look closely at the axes, and we can see that the size has been halved from the 500's to the 250's.

![Walking-through-convolutions-6](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-6.png)

For fun, we can try the other filter, run it, and then compare the convolution with its pooled version. Again, we can see that the features have not just been maintained, they may have also been emphasized a bit. 

![Walking-through-convolutions-7](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Walking-through-convolutions-7.png)

## Week4: Using Real-world Images

### Understanding ImageDataGenerator

One feature of the image generator is that you can point it at a directory and then the sub-directories of that will automatically generate labels for you.（你可以将图片生成器指向一个目录，然后这个目录的子目录会自动生成标签。）

例如下图Training和Validation目录下分别有Hourses和Humans目录，只要图片生成器指定Training或Validation目录，图片就会以子目录Hourses和Humans自动生成标签。

![Understanding-ImageDataGenerator-1](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Understanding-ImageDataGenerator-1.png)

![Understanding-ImageDataGenerator-2](https://gitlab.com/iknowledge/CourseImage/-/raw/main/Coursera/TensorFlowDeveloper/Understanding-ImageDataGenerator-2.png)

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

train_datagen = ImageDataGenerator(rescale=1./255)

train_generator = train_datagen.flow_from_directory(
    train_dir,
    target_size=(300,300),
    batch_size=128,
    class_mode='binary'
)

test_datagen = ImageDataGenerator(rescale=1/.255)

validation_generator = test_datagen.flow_from_directory(
    validation_dir,
    target_size=(300, 300),
    batch_size=32,
    class_mode='binary'
)
```

I'm going to pass rescale to it to normalize the data.（传递rescale参数进行数据标准化。）

You can then call the flow_from_directory method on it to get it to load images from that directory and its sub-directories.（调用flow_from_directory方法从指定目录加载数据。）

You should always point it at the directory that contains sub-directories that contain your images. The names of the sub-directories will be the labels for your images that are contained within them.（第一个参数必须指定包含图片文件夹的父目录，包含图片的子目录的名称就是图片的标签。）

Now, images might come in all shapes and sizes and unfortunately for training a neural network, the input data all has to be the same size, so the images will need to be resized to make them consistent.（target_size用于调整图片大小使其一致，因为现实中图片可能大小形状各异，而输入的图片必须是相同大小的。）

The images will be loaded for training and validation in batches where it's more efficient than doing it one by one.（batch_size指定批次大小，因为按批次加载图片进行训练和验证比一张一张加载更高效。）

Finally, there's the class mode. Now, this is a binary classifier i.e. it picks between two different things; horses and humans.（class_mode指定一个二元分类器。）

### Defining a ConvNet to use complex images

```python
model = tf.keras.models.Sequential([
    tf.keras.layers.Conv2D(16, (3,3), activation='relu',input_shape=(300, 300, 3)),
    tf.keras.layers.MaxPooling2D(2, 2),
    tf.keras.layers.Conv2D(32, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2), tf.keras.layers.Flatten(),
    tf.keras.layers.Dense(512, activation='relu'),
    tf.keras.layers.Dense(1, activation='sigmoid')
])
```

You'll notice that there are three sets of convolution pooling layers at the top. This reflects the higher complexity and size of the images.（顶部有三组卷积池化层，这反应了图片有更高的复杂度和尺寸。）

Another thing to pay attention to is the input shape. We resize their images to be 300 by 300 as they were loaded, but they're also color images. So there are three bytes per pixel. One byte for the red, one for green, and one for the blue channel, and that's a common 24-bit color pattern.（输入形状是300x300的彩色图片，所有每个像素点有三个字节，每个字节分别代表红、绿、蓝三种通道。）

Remember before when you created the output layer, you had one neuron per class, but now there's only one neuron for two classes. That's because we're using a different activation function where sigmoid is great for binary classification, where one class will tend towards zero and the other class tending towards one.（你之前创建的输出层，每个类有一个神经元，但是现在两个类只有一个神经元。这是因为我们使用了sigmoid作为激活函数，sigmoid更擅长做二分类，一个类趋向于0，另一个类趋向于1。）

Now, if we take a look at our model summary, we can see the journey of the image data through the convolutions.（现在通过模型的summary方法查看图片数据通过卷积的过程。）

```
_________________________________________________________________
 Layer (type)                Output Shape              Param #   
=================================================================
 conv2d_5 (Conv2D)           (None, 298, 298, 16)      448       
                                                                 
 max_pooling2d_5 (MaxPooling  (None, 149, 149, 16)     0         
 2D)                                                             
                                                                 
 conv2d_6 (Conv2D)           (None, 147, 147, 32)      4640      
                                                                 
 max_pooling2d_6 (MaxPooling  (None, 73, 73, 32)       0         
 2D)                                                             
                                                                 
 conv2d_7 (Conv2D)           (None, 71, 71, 64)        18496     
                                                                 
 max_pooling2d_7 (MaxPooling  (None, 35, 35, 64)       0         
 2D)                                                             
                                                                 
 flatten_1 (Flatten)         (None, 78400)             0         
                                                                 
 dense_2 (Dense)             (None, 512)               40141312  
                                                                 
 dense_3 (Dense)             (None, 1)                 513       
                                                                 
=================================================================
Total params: 40,165,409
Trainable params: 40,165,409
Non-trainable params: 0
_________________________________________________________________
```

The 300 by 300 becomes 298 by 298 after the three by three filter, it gets pulled to 149 by 149 which in turn gets reduced to 73 by 73 after the filter that then gets pulled to 35 by 35, this will then get flattened, so 64 convolutions that are 35 squared and shape will get fed into the DNN.（300x300的图片通过3x3的过滤器变成298x298，再经过2x2的池化层变成149x149，接着通过同样的卷积减少到73x73，又变成35x35，然后变扁平，所以64个形状为35x35的卷积将被送入DNN。）

If you multiply 35 by 35 by 64, you get 78,400, and that's the shape of the data once it comes out of the convolutions.（35x35x64=78400，这就是卷积后的数据形状。）

### Training the ConvNet

```python
from tensorflow.keras.optimizers import RMSprop

model.compile(loss='binary_crossentropy',
            optimizer=RMSprop(lr=0.001),
            metrics=['accuracy'])
```

Because we're doing a binary choice here, let's pick a binary_crossentropy instead.

I thought it would be fun to use the RMSprop, where you can adjust the learning rate to experiment with performance.

```python
history = model.fit(
    train_generator,
    steps_per_epoch=8,
    epochs=15,
    validation_data=validation_generator,
    validation_steps=8,
    verbose=2)
```

The first parameter is the training generator that you set up earlier. This streams the images from the training directory.（第一个参数是您之前设置的训练集生成器。它将从训练目录流式传输图像。）

There are 1,024 images in the training directory, so we're loading them in 128 at a time. So in order to load them all, we need to do 8 batches. So we set the steps_per_epoch to cover that.（训练目录中有1,024张图片，因此我们每次加载128张。因此，为了全部加载这些图像，我们需要进行8次分批加载。所以我们设置了steps_per_epoch来满足这个要求。）

Now we specify the validation set that comes from the validation_generator that we also created earlier. It had 256 images, and we wanted to handle them in batches of 32, so we will do 8 steps.（现在我们指定验证集，它来自我们之前创建的validation_generator。它有256张图像，我们希望以32张为一批进行处理，因此我们将执行8步。）

The verbose parameter specifies how much to display while training is going on. With verbose set to 2, we'll get a little less animation hiding the epoch progress.（verbose参数指定在训练过程中显示多少内容。当verbose设置为2时，我们将获得较少的隐藏epoch进度的动画。）

```python
import numpy as np
from google.colab import files
from keras.preprocessing import image

uploaded = files.upload()
for fn in uploaded.keys():
    # predicting images
    path = '/content/' + fn
    img = image.load_img(path, target_size=(300, 300))
    x = image.img_to_array(img)
    x = np.expand_dims(x, axis=0)
    images = np.vstack([x])
    classes = model.predict(images, batch_size=10)
    print(classes[0])
    if classes[0]>0.5:
        print(fn + " is a human")
    else:
        print(fn + " is a horse")
```

So these parts are specific to Colab, they are what gives you the button that you can press to pick one or more images to upload. The image paths then get loaded into this list called uploaded.(files.upload()是Colab特有的，它们为您提供了一个按钮，您可以按这个按钮来选择一张或多张要上传的图片)

The loop then iterates through all of the images in that collection. And you can load an image and prepare it to input into the model with this code. Take note to ensure that the dimensions match the input dimensions that you specified when designing the model.（注意确保图片尺寸与你在设计模型时指定的输入尺寸一致。）

You can then call model.predict, passing it the details, and it will return an array of classes. In the case of binary classification, this will only contain one item with a value close to 0 for one class and close to 1 for the other.（调用model.predict，将详细信息传递给它，它将返回一个类别数组。在二元分类的情况下，该数组将只包含一个值接近0的项，而另一个值接近1的项）

### Training the neural network

Binary Crossentropy: https://gombru.github.io/2018/05/23/cross_entropy_loss/

RMSProp: http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf
