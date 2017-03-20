
```python
import numpy as np 
#from keras.datasets import mnist
from keras.models import Sequential
from keras.layers.core import Dense, Dropout, Activation, Flatten
from keras.optimizers import SGD, Adam, RMSprop
from keras.utils import np_utils
from keras.layers import Convolution2D, MaxPooling2D
```

    Using Theano backend.
    Using gpu device 0: GeForce GTX 750 Ti (CNMeM is disabled, cuDNN not available)



```python
import pickle
with open("/home/sdyang/Desktop/LIDC/exports/benign_2.5.p","rb") as input_data:
    (benign_data, benign_label)=pickle.load(input_data, encoding='bytes')
with open("/home/sdyang/Desktop/LIDC/exports/malign_2.5.p","rb") as input_data:
    (malignanat_data, malignanat_label)=pickle.load(input_data, encoding='bytes')
    
with open("/home/sdyang/Desktop/LIDC/exports/benign_val_2.5.p","rb") as input_data:
    (benign_data_val, benign_label_val)=pickle.load(input_data, encoding='bytes')
with open("/home/sdyang/Desktop/LIDC/exports/malign_val_2.5.p","rb") as input_data:
    (malignanat_data_val, malignanat_label_val)=pickle.load(input_data, encoding='bytes')
```


```python
indexset=np.arange(len(benign_data))
nb_validationSamples=17000

selected=np.random.choice(indexset,nb_validationSamples,replace=False)
selected_benign_data=benign_data[selected]
selected_benign_label=benign_label[selected]
```


```python
X_train=np.vstack((selected_benign_data,malignanat_data))
X_test=np.vstack((benign_data_val,malignanat_data_val))

y_train=np.append(selected_benign_label,malignanat_label)
y_test=np.append(benign_label_val,malignanat_label_val)
```


```python
#to use theano with gpu
new_y_train=[]
new_y_test=[]
for i in y_train:
    new_y_train.append(int(i))
    
for i in y_test:
    new_y_test.append(int(i))
    
new_y_tr=np.array(new_y_train)
new_y_te=np.array(new_y_test)
```


```python
#these loaded data is np.array in default
#X_train = X_train.reshape(60000, 28, 28, 1)
#X_test = X_test.reshape(10000, 28, 28, 1)
X_train = X_train.reshape(X_train.shape[0], 50, 50, 1)
X_test = X_test.reshape(X_test.shape[0], 50, 50, 1)


#change dtype into float32
X_train = X_train.astype('float32')
X_test = X_test.astype('float32')
X_train /= 1000
X_test /= 1000
print(X_train.shape[0], 'train samples')
print(X_test.shape[0], 'test samples')


#convert class vectors to binary class matrices
nb_classes=2
#Y_train = np_utils.to_categorical(y_train, nb_classes)
#Y_test = np_utils.to_categorical(y_test, nb_classes)
Y_train = np_utils.to_categorical(new_y_tr, nb_classes)
Y_test = np_utils.to_categorical(new_y_te, nb_classes)
```

    34499 train samples
    791 test samples



```python
from keras import backend as K
K.set_image_dim_ordering('tf')
```


```python
##################################################
# set parameters to use CNN

# number of convolutional filters to use
nb_filters = 32
# size of pooling area for max pooling
pool_size = (2, 2)
# convolution kernel size
kernel_size = (3, 3)
# input_shape. At first, i set the input_shape as (1, 28, 28). however, after that, i can't apply fully connected layer.
input_shape=(50, 50, 1)

model=Sequential()

#input_shape :(sizeofwith, sizeofheight, 1) when we use grayscale image(1 channel). If we want to use RGB image use (sizeofwith, sizeofheight, 1)

model.add(Convolution2D(64, kernel_size[0], kernel_size[1],
                        border_mode='valid',
                        input_shape=input_shape, name='conv1'))
model.add(Activation('relu', name='act1'))

model.add(MaxPooling2D(pool_size=pool_size, name='pool1'))
model.add(Convolution2D(64, kernel_size[0], kernel_size[1],
                        border_mode='valid', name='conv2'))
model.add(Activation('relu', name='act2'))
model.add(MaxPooling2D(pool_size=pool_size, name='pool2'))
model.add(Convolution2D(64, kernel_size[0], kernel_size[1],
                        border_mode='valid', name='conv3'))
model.add(Activation('relu', name='act3'))
model.add(MaxPooling2D(pool_size=pool_size, name='pool3'))
model.add(Dropout(0.25, name='dropout1'))
#flatten makes multidimensional weights into 1-d.
#_________________
#dropout_15 (Dropout)             (None, 13, 13, 32)    0           maxpooling2d_16[0][0]
#____________________________________________________________________________________________________
#flatten_14 (Flatten)             (None, 5408)          0           dropout_15[0][0]
#_
model.add(Convolution2D(150, 4, 4,
                        border_mode='valid', name='conv4'))
model.add(Activation('relu', name='act4'))
model.add(Convolution2D(2, 1, 1,
                        border_mode='valid', name='conv5'))
model.add(Activation('sigmoid', name='act5'))

model.summary()

#sgd=SGD()
sgd=SGD(lr=0.1, momentum=0.9, decay=0.0, nesterov=False)

#the command 'show_accuracy' is deprecated. Instead of it, use metrics argument when we compile the model.

model.compile(loss='mean_squared_error', optimizer='RMSprop', metrics=['accuracy']) 
#model.compile(loss='binary_crossentropy', optimizer='RMSprop', metrics=['accuracy']) 
```

    ____________________________________________________________________________________________________
    Layer (type)                     Output Shape          Param #     Connected to                     
    ====================================================================================================
    conv1 (Convolution2D)            (None, 48, 48, 64)    640         convolution2d_input_2[0][0]      
    ____________________________________________________________________________________________________
    act1 (Activation)                (None, 48, 48, 64)    0           conv1[0][0]                      
    ____________________________________________________________________________________________________
    pool1 (MaxPooling2D)             (None, 24, 24, 64)    0           act1[0][0]                       
    ____________________________________________________________________________________________________
    conv2 (Convolution2D)            (None, 22, 22, 64)    36928       pool1[0][0]                      
    ____________________________________________________________________________________________________
    act2 (Activation)                (None, 22, 22, 64)    0           conv2[0][0]                      
    ____________________________________________________________________________________________________
    pool2 (MaxPooling2D)             (None, 11, 11, 64)    0           act2[0][0]                       
    ____________________________________________________________________________________________________
    conv3 (Convolution2D)            (None, 9, 9, 64)      36928       pool2[0][0]                      
    ____________________________________________________________________________________________________
    act3 (Activation)                (None, 9, 9, 64)      0           conv3[0][0]                      
    ____________________________________________________________________________________________________
    pool3 (MaxPooling2D)             (None, 4, 4, 64)      0           act3[0][0]                       
    ____________________________________________________________________________________________________
    dropout1 (Dropout)               (None, 4, 4, 64)      0           pool3[0][0]                      
    ____________________________________________________________________________________________________
    conv4 (Convolution2D)            (None, 1, 1, 150)     153750      dropout1[0][0]                   
    ____________________________________________________________________________________________________
    act4 (Activation)                (None, 1, 1, 150)     0           conv4[0][0]                      
    ____________________________________________________________________________________________________
    conv5 (Convolution2D)            (None, 1, 1, 2)       302         act4[0][0]                       
    ____________________________________________________________________________________________________
    act5 (Activation)                (None, 1, 1, 2)       0           conv5[0][0]                      
    ====================================================================================================
    Total params: 228548
    ____________________________________________________________________________________________________



```python
Y_test=Y_test.reshape(len(Y_test),1,1,2)
Y_train=Y_train.reshape(len(Y_train),1,1,2)
```


```python
nb_epochs=30
for i in range(nb_epochs):
    h=model.fit(X_train,Y_train, batch_size=128, shuffle=True, nb_epoch=1, verbose=1, validation_data=(X_test,Y_test))
    filename='weights'+str(i+1)+'.hdf5'
    model.save_weights(filename,overwrite=True)
    
    print(str(i+1)+'th epoch is done!')
    print("="*50)
```

    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.2123 - acc: 0.6647 - val_loss: 0.2075 - val_acc: 0.7004
    1th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.1625 - acc: 0.7636 - val_loss: 0.2135 - val_acc: 0.7029
    2th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.1359 - acc: 0.8058 - val_loss: 0.2082 - val_acc: 0.6991
    3th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.1193 - acc: 0.8327 - val_loss: 0.1893 - val_acc: 0.7295
    4th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.1041 - acc: 0.8561 - val_loss: 0.2146 - val_acc: 0.6890
    5th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0903 - acc: 0.8774 - val_loss: 0.2003 - val_acc: 0.7295
    6th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0808 - acc: 0.8914 - val_loss: 0.2225 - val_acc: 0.7080
    7th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0722 - acc: 0.9041 - val_loss: 0.2150 - val_acc: 0.7181
    8th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 62s - loss: 0.0642 - acc: 0.9162 - val_loss: 0.2298 - val_acc: 0.7029
    9th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0569 - acc: 0.9256 - val_loss: 0.2308 - val_acc: 0.7092
    10th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0504 - acc: 0.9359 - val_loss: 0.2513 - val_acc: 0.6726
    11th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0464 - acc: 0.9399 - val_loss: 0.2398 - val_acc: 0.6877
    12th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0416 - acc: 0.9457 - val_loss: 0.2278 - val_acc: 0.7332
    13th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0366 - acc: 0.9530 - val_loss: 0.2558 - val_acc: 0.6903
    14th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 62s - loss: 0.0340 - acc: 0.9570 - val_loss: 0.2422 - val_acc: 0.6953
    15th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0306 - acc: 0.9610 - val_loss: 0.2587 - val_acc: 0.6814
    16th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0279 - acc: 0.9651 - val_loss: 0.2695 - val_acc: 0.6675
    17th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0257 - acc: 0.9672 - val_loss: 0.2645 - val_acc: 0.6827
    18th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0234 - acc: 0.9704 - val_loss: 0.2740 - val_acc: 0.6662
    19th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0214 - acc: 0.9729 - val_loss: 0.2799 - val_acc: 0.6789
    20th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0205 - acc: 0.9745 - val_loss: 0.2693 - val_acc: 0.6827
    21th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0187 - acc: 0.9758 - val_loss: 0.2662 - val_acc: 0.6865
    22th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0181 - acc: 0.9769 - val_loss: 0.2829 - val_acc: 0.6713
    23th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0179 - acc: 0.9776 - val_loss: 0.2619 - val_acc: 0.6979
    24th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0157 - acc: 0.9805 - val_loss: 0.2617 - val_acc: 0.6928
    25th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0151 - acc: 0.9814 - val_loss: 0.2729 - val_acc: 0.6827
    26th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0139 - acc: 0.9825 - val_loss: 0.2752 - val_acc: 0.6852
    27th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0132 - acc: 0.9827 - val_loss: 0.2826 - val_acc: 0.6738
    28th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0121 - acc: 0.9850 - val_loss: 0.2699 - val_acc: 0.6928
    29th epoch is done!
    ==================================================
    Train on 34499 samples, validate on 791 samples
    Epoch 1/1
    34499/34499 [==============================] - 61s - loss: 0.0117 - acc: 0.9854 - val_loss: 0.2952 - val_acc: 0.6688
    30th epoch is done!
    ==================================================



```python
model.load_weights('/home/sdyang/Desktop/pythonfiles/LIDC/disc_records/try7_65535_mse/weights13.hdf5')
```


```python
prediction=model.predict(X_test[0:473,:,:])
predictedlabel=np.argmax(prediction,axis=3)
```


```python
prediction2=model.predict(X_test[473:,:,:])
predictedlabel2=np.argmax(prediction,axis=3)
```


```python
len(prediction2)
```




    318




```python
prediction2[:,0,0,0]>0.5
```




    array([False,  True, False,  True,  True,  True,  True,  True,  True,
            True, False, False, False, False, False,  True, False,  True,
            True,  True,  True, False,  True,  True,  True,  True,  True,
            True,  True,  True,  True,  True,  True,  True,  True,  True,
           False, False, False,  True,  True,  True,  True,  True,  True,
            True,  True, False, False, False, False, False, False, False,
           False, False, False, False, False, False, False, False, False,
            True,  True,  True,  True, False, False, False, False, False,
           False, False, False, False,  True,  True, False,  True,  True,
           False, False, False, False,  True,  True,  True,  True,  True,
            True, False, False, False, False, False, False, False, False,
           False, False, False, False, False, False,  True,  True,  True,
            True,  True,  True,  True,  True,  True,  True,  True,  True,
            True, False, False,  True, False, False,  True,  True,  True,
            True,  True, False, False, False, False, False, False, False,
            True, False, False, False,  True, False, False, False, False,
           False,  True,  True,  True,  True,  True,  True, False, False,
           False, False, False, False, False, False, False, False, False,
           False,  True,  True,  True, False, False, False,  True,  True,
           False,  True,  True, False,  True, False, False, False, False,
           False, False,  True, False, False,  True, False, False, False,
           False, False, False, False, False, False, False, False, False,
           False, False,  True, False, False, False, False, False, False,
           False, False, False, False, False, False, False, False, False,
           False, False, False, False, False, False, False, False,  True,
            True,  True,  True,  True, False, False, False, False, False,
           False, False, False, False, False,  True,  True,  True, False,
            True, False, False, False, False, False, False, False, False,
           False,  True,  True,  True, False, False, False, False, False,
            True,  True,  True,  True,  True,  True, False, False, False,
           False, False, False, False, False, False, False,  True,  True,
            True,  True,  True,  True,  True, False, False, False, False,
           False, False, False, False, False, False,  True,  True,  True,
            True,  True, False,  True, False, False, False, False, False,
            True, False,  True,  True,  True, False, False, False,  True,
            True,  True,  True], dtype=bool)




```python
len(np.where(prediction2[:,0,0,0]<0.5)[0])
```




    192




```python
len(np.where(prediction[:,0,0,0]>0.5)[0])
```




    391




```python
#the last probability value
prediction=model.predict(X_test, verbose=1)
predictedlabel=np.argmax(prediction,axis=1)
```


```python
prediction[:,0,0,0]
```




    array([  2.28516664e-03,   9.34649445e-03,   9.84741926e-01,
             9.76729274e-01,   9.76624191e-01,   2.02211813e-05,
             8.02494679e-03,   9.99909520e-01,   9.72897828e-01,
             9.98100936e-01], dtype=float32)



TEST


```python
from sklearn.preprocessing import StandardScaler
from sklearn.preprocessing import MinMaxScaler
```


```python
StandardScaler?
```


```python
scaler=MinMaxScaler(feature_range=(0,1))
scaler.fit_transform?
```


```python
Wj=model.get_weights()
#model.weights=
#[convolution2d_1_W,
# convolution2d_1_b,
# dense_1_W,
# dense_1_b,
# dense_2_W,
# dense_2_b]

W0=Wj[0]
W1=Wj[1]

W2=Wj[2]
W3=Wj[3]
W4=Wj[4]
W5=Wj[5]

#the number of parameters in model.summary is the nb of free parameters.
```


```python
#the use of multithreads in cpus
import theano
theano.config.openmp= 'True'
```


```python
#the use of multithreads in cpus
import theano
theano.config.openmp= 'False'
```


```python

```