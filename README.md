# CONV-Neural-Network



pip install -q tensorflow tensorflow-datasets

import matplotlib as plt
import tensorflow as tf
import tensorflow_datasets as tfds
from tensorflow import keras
import numpy as np

builder = tfds.builder('rock_paper_scissors')
info = builder.info
info

df_train = tfds.load(name="rock_paper_scissors",split = 'train')
df_test = tfds.load(name='rock_paper_scissors',split = 'test')

fig = tfds.show_examples(info,df_train)

train_images = np.array([example['image'].numpy()[:,:,0] for example in df_train])
train_labels = np.array([example['label'].numpy() for example in df_train])
test_images = np.array([example['image'].numpy()[:,:,0] for example in df_test])
test_labels = np.array([example['label'].numpy() for example in df_test])

train_images = train_images.reshape(2520,300,300,1)
test_images = test_images.reshape(372,300,300,1)
train_images = train_images.astype('float32')
test_images = test_images.astype('float32')
train_images/=255
test_images/=255

data_augmentation = tf.keras.Sequential([
  keras.layers.experimental.preprocessing.RandomFlip("horizontal_and_vertical"),
  keras.layers.experimental.preprocessing.RandomRotation(0.3),
])



IMG_SIZE = 700

resize_and_rescale = tf.keras.Sequential([
  keras.layers.experimental.preprocessing.Resizing(IMG_SIZE, IMG_SIZE),
  keras.layers.experimental.preprocessing.Rescaling(1./255)
])

model = keras.Sequential([data_augmentation,
                          keras.layers.MaxPooling2D(pool_size=(4,4),strides=3,input_shape=(300,300,1)),
                          keras.layers.Conv2D(60, 3,strides=2,activation='relu'),
                          keras.layers.Conv2D(30,3,strides=2,activation='relu'),
                          keras.layers.MaxPooling2D(pool_size=(3,3),strides=2),
                          keras.layers.Dropout(0.5),
                          keras.layers.MaxPooling2D(pool_size=(2,2),strides=2),
                          keras.layers.Flatten(),
                          keras.layers.Dense(128,activation='relu'),
                          keras.layers.Dense(3,activation='softmax'),

                          ])
#Try passing in a pooling matrix for diagonal lines


model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])


model.fit(train_images,train_labels,epochs=5)

model.evaluate(test_images,test_labels)
