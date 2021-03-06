---
layout:     post  
title:      深度神经网络编程作业(一)  
keywords:   博客   
categories: [机器学习]   
tags:	    [深度学习，深层神经网络]  
---

这部分是深层神经网络的编程作业的上篇，主要是初始化和前向传播的实现。

## 1 - 需要的库文件  

Let's first import all the packages that you will need during this assignment.          

-  [numpy](www.numpy.org) 是Python进行科学计算的主要库文件。
-  [matplotlib](http://matplotlib.org) 是Python的2D绘图库，仅需要几行代码，便可以生成出版质量级别的图形。    
- dnn_utils 包含了这次作业需要用到的一些函数定义。  
- testCases 包含了用于验证提交代码正确性的测试用例。
- np.random.seed(1) 用于给作业打分的，请不要改变seed()值。seed()用于指定随机数生成时所用算法开始的整数值，如果使用相同的seed()值，则每次生成的随机数都相同，如果不设置这个值，则系统根据时间自己选择这个值，此时每次生成的随机数因时间差异而不同。 


    import numpy as np   
	import h5py   
	import matplotlib.pyplot as plt    
	from testCases\_v4 import *    
	from dnn\_utils\_v2 import sigmoid, sigmoid_backward, relu, relu\_backward    
	
	%matplotlib inline     
	plt.rcParams['figure.figsize'] = (5.0, 4.0) # set default size of plots   
	plt.rcParams['image.interpolation'] = 'nearest'      
	plt.rcParams['image.cmap'] = 'gray'      
	
	%load_ext autoreload
	%autoreload 2
	
	np.random.seed(1)

## 2 - 作业大纲

你需要先完成一些辅助函数的编码（会提供详细的说明），这些函数会帮助你完成构建深度神经网络。以下是作业大纲：

- 参数初始化。
- 完成前向传播模块（下图的紫色部分）。
     - 完成前向传播的线性模块(resulting in  $Z^{[l]}$)。
     - 我们提供给你激励函数 (relu/sigmoid)。
     - 结合前面两步得到 [LINEAR->ACTIVATION] 的前向传播函数。
     - 执行 [LINEAR->RELU] 前向传播函数 L-1 次 ( 1层 至 L-1层) 且 执行一次 [LINEAR->SIGMOID]  (**L** 层)。最终你就得到了L层模型的前向传播函数。   
- 计算损失。
- 完成反向传播模块（下图的红色部分）.
    - 完成反向传播的线性部分。
    - 我们提供给你激励函数(relu_backward/sigmoid_backward) 的梯度下降算法。 
    - 结合前面两步得到 [LINEAR->ACTIVATION] 的反向传播函数。
    - 执行 [LINEAR->RELU] 反向传播函数 L-1 次 且 执行一次[LINEAR->SIGMOID] 反向传播函数。最终即是L层模型的反向传播函数。
- 更新参数。  

   ![](/images/images_2018/final outline.png)         
       
 <center>**Figure 1**</center> 


**Note** 每一个前向传播的函数，都会对应一个后向传播的函数，因此你需要存储一些有用的值，在后向传播中用来计算梯度下降。作业完整显示了如何具体实现这些步骤。

## 3 - 初始化

你需要编写两个辅助函数来初始化模型的参数。一个用来初始化2层模型的参数，一个用来初始化**L**层模型的参数。

### 3.1 - 2层神经网络

**Exercise**: 创建并初始化2层神经网络的参数。

**Instructions**:   
- 模型结构: *LINEAR -> RELU -> LINEAR -> SIGMOID*.      
- 权重矩阵使用随机初始化： Use `np.random.randn(shape)*0.01` with the correct shape.   
- 偏差使用0初始化： Use `np.zeros(shape)`.
     
	# GRADED FUNCTION: initialize_parameters
	
	def initialize_parameters(n_x, n_h, n_y):
	    """
	    Argument:
	    n_x -- size of the input layer
	    n_h -- size of the hidden layer
	    n_y -- size of the output layer
	    
	    Returns:
	    parameters -- python dictionary containing your parameters:
	                    W1 -- weight matrix of shape (n_h, n_x)
	                    b1 -- bias vector of shape (n_h, 1)
	                    W2 -- weight matrix of shape (n_y, n_h)
	                    b2 -- bias vector of shape (n_y, 1)
	    """
	    
	    np.random.seed(1)
	    
	    ### START CODE HERE ### (≈ 4 lines of code)
	    W1 = np.random.randn(n_h,n_x) * 0.01
	    b1 = np.zeros((n_h,1))
	    W2 = np.random.randn(n_y,n_h) * 0.01
	    b2 = np.zeros((n_y,1))
	    ### END CODE HERE ###
	    
	    assert(W1.shape == (n_h, n_x))
	    assert(b1.shape == (n_h, 1))
	    assert(W2.shape == (n_y, n_h))
	    assert(b2.shape == (n_y, 1))
	    
	    parameters = {"W1": W1,
	                  "b1": b1,
	                  "W2": W2,
	                  "b2": b2}
	    
	    return parameters    


		parameters = initialize_parameters(3,2,1)
		print("W1 = " + str(parameters["W1"]))
		print("b1 = " + str(parameters["b1"]))
		print("W2 = " + str(parameters["W2"]))
		print("b2 = " + str(parameters["b2"]))

-----------------------------------

**Expected output**:
       
<table style="width:80%">
  <tr>
    <td> W1 </td>
    <td> [[ 0.01624345 -0.00611756 -0.00528172]
 [-0.01072969  0.00865408 -0.02301539]] </td> 
  </tr>

  <tr>
    <td> b1 </td>
    <td>[[ 0.]
 [ 0.]]</td> 
  </tr>
  
  <tr>
    <td> W2 </td>
    <td> [[ 0.01744812 -0.00761207]]</td>
  </tr>
  
  <tr>
    <td> b2 </td>
    <td> [[ 0.]] </td> 
  </tr>
</table>

### np.random.randn()  

np.random.randn(d0,d1,...,dn)  

- d0,d1,...,dn表示返回的array的维度，当没有参数时，返回float   
- 返回一个或一组具有标准正态分布的样本，返回值类型为ndarray or float   

        np.random.randn(2,1)   

执行结果：  
        
    array([[0.41230963],
       [1.76974892]]) 

### np.zeros()  

np.zeros(shape, dtype=float, order='C')  

- shape表示返回的array的形状，比如(2,3) or 2 
- dtype表示返回array中的数据的类型，默认是np.float64 
- 返回指定形状的ndarray

        np.zeros((2,1))

执行结果：  

	array([[0.],
	       [0.]])  


### 3.2 - L层神经网络

对于深层神经网络，初始化更复杂，因为有更多的权重矩阵和偏差向量。 当完成 `initialize_parameters_deep`时，你需要保证各层的维度的准确性。


记住，在Python中计算 $W X + b$ 时, 会用到Broadcasting机制（将在另一篇博客中专门来阐述它）。 

**Exercise**: 完成L层神经网络的初始化。    


**Instructions**:   
    
- 模型结构：  *[LINEAR -> RELU] $ \times$ (L-1) -> LINEAR -> SIGMOID*。即 **L-1** 层使用ReLU激励函数，**L**（输出层）使用sigmoid 激励函数。    
- 权重矩阵使用随机初始化：Use `np.random.randn(shape) * 0.01`。  
- 偏差使用0初始化：Use `np.zeros(shape)`。    
- 使用变量`layer_dims`存储各层的神经单元数，比如[2,4,1]表示：输入层2个单元，隐藏层4个单元，输出层1个单元。因此`W1`的shape是 (4,2)， `b1` 是 (4,1), `W2` 是 (1,4) ， `b2` 是 (1,1)。 你可以扩展到L层的情况。    
- 这里给出了L=1时的具体实现，你可以依此得到L层的实现方法。


    if L == 1:
        parameters["W" + str(L)] = np.random.randn(layer_dims[1], layer_dims[0]) * 0.01
        parameters["b" + str(L)] = np.zeros((layer_dims[1], 1))

-------------------------------------------------------

	# GRADED FUNCTION: initialize_parameters_deep
	
	def initialize_parameters_deep(layer_dims):
	    """
	    Arguments:
	    layer_dims -- python array (list) containing the dimensions of each layer in our network
	    
	    Returns:
	    parameters -- python dictionary containing your parameters "W1", "b1", ..., "WL", "bL":
	                    Wl -- weight matrix of shape (layer_dims[l], layer_dims[l-1])
	                    bl -- bias vector of shape (layer_dims[l], 1)
	    """
	    
	    np.random.seed(3)
	    parameters = {}
	    L = len(layer_dims)            # number of layers in the network
	
	    for l in range(1, L):
	        ### START CODE HERE ### (≈ 2 lines of code)
	        parameters['W' + str(l)] = np.random.randn(layer_dims[l], layer_dims[l-1]) * 0.01
	        parameters['b' + str(l)] = np.zeros((layer_dims[l],1))
	        ### END CODE HERE ###
	        
	        assert(parameters['W' + str(l)].shape == (layer_dims[l], layer_dims[l-1]))
	        assert(parameters['b' + str(l)].shape == (layer_dims[l], 1))
	
	        
	    return parameters

	parameters = initialize_parameters_deep([5,4,3])
	print("W1 = " + str(parameters["W1"]))
	print("b1 = " + str(parameters["b1"]))
	print("W2 = " + str(parameters["W2"]))
	print("b2 = " + str(parameters["b2"]))

**Expected output**:
       
<table style="width:80%">
  <tr>   
    <td> W1 </td>
    <td> 
 [[ 0.01788628  0.0043651   0.00096497 -0.01863493 -0.00277388]
 [-0.00354759 -0.00082741 -0.00627001 -0.00043818 -0.00477218]   
 [-0.01313865  0.00884622  0.00881318  0.01709573  0.00050034]
 [-0.00404677 -0.0054536  -0.01546477  0.00982367 -0.01101068]]</td> 
  </tr>
  
  <tr>
    <td>b1 </td>
    <td>[[ 0.]
 [ 0.]
 [ 0.]
 [ 0.]]</td> 
  </tr>
  
  <tr>
    <td>W2 </td>
    <td>
[[-0.01185047 -0.0020565   0.01486148  0.00236716]
 [-0.01023785 -0.00712993  0.00625245 -0.00160513]
 [-0.00768836 -0.00230031  0.00745056  0.01976111]]</td> 
  </tr>
  
  <tr>
    <td>b2 </td>
    <td>[[ 0.]
 [ 0.]
 [ 0.]]</td> 
  </tr>
  
</table>  




## 4 - 前向传播模块

### 4.1 - Linear Forward  

我们将依次实现3个函数的编写:
 
- LINEAR
- LINEAR -> ACTIVATION where ACTIVATION will be either ReLU or Sigmoid. 
- [LINEAR -> RELU] $\times$ (L-1) -> LINEAR -> SIGMOID (whole model)

这个部分即以下公式的向量化实现：  

$$Z^{[l]} = W^{[l]}A^{[l-1]} +b^{[l]}\tag{4}$$

where $A^{[0]} = X$. 

**Exercise**: 完成前向传播算法的线性部分。

**Reminder**: 

需要用到的数学公式是: $Z^{[l]} = W^{[l]}A^{[l-1]} +b^{[l]}$。 函数`np.dot()` 会很有用。 你需要保证各个变量的维度正确性，`W.shape`能帮助查看相关信息。   


	# GRADED FUNCTION: linear_forward
	
	def linear_forward(A, W, b):
	    """
	    Implement the linear part of a layer's forward propagation.
	
	    Arguments:
	    A -- activations from previous layer (or input data): (size of previous layer, number of examples)
	    W -- weights matrix: numpy array of shape (size of current layer, size of previous layer)
	    b -- bias vector, numpy array of shape (size of the current layer, 1)
	
	    Returns:
	    Z -- the input of the activation function, also called pre-activation parameter 
	    cache -- a python dictionary containing "A", "W" and "b" ; stored for computing the backward pass efficiently
	    """
	    
	    ### START CODE HERE ### (≈ 1 line of code)
	    Z = np.dot(W,A) + b
	    ### END CODE HERE ###
	    
	    assert(Z.shape == (W.shape[0], A.shape[1]))
	    cache = (A, W, b)
	    
	    return Z, cache

-------------------------

	A, W, b = linear_forward_test_case()
	Z, linear_cache = linear_forward(A, W, b)
	print("Z = " + str(Z))

---------------------------

**Expected output**:

<table style="width:35%">
  
  <tr>
    <td> Z </td>
    <td> [[ 3.26295337 -1.23429987]] </td> 
  </tr>
  
</table>

### np.dot()  

对于二维矩阵，是计算矩阵的乘积

	A = np.array([[1,2,3],[4,5,6]])
	B = np.array([[1,2],[3,4],[5,6]])
	np.dot(A,B)
执行结果：
 
	array([[22, 28],
	       [49, 64]])

对于一维矩阵，则是计算两者的内积     

	C = np.array([1,2,3])
	D = np.array([4,5,6])
	np.dot(C,D)

执行结果：   

	32


### 4.2 - Linear-Activation Forward

这次作业会用到两个激励函数： 

- **Sigmoid**: $\sigma(Z) = \sigma(W A + b) = \frac{1}{ 1 + e^{-(W A + b)}}$. 作业里已经实现了 `sigmoid` 函数。函数返回了两个项： A 和 包含 "`Z`" 的`cache`，在作业里直接调用该函数即可：
``` 
A, activation_cache = sigmoid(Z)
```

- **ReLU**: ReLu的数学公式是 $A = RELU(Z) = max(0, Z)$。同样的，`relu` 函数已经实现了，返回项同`sigmoid` 函数，在作业里直接调用该函数即可：
``` 
A, activation_cache = relu(Z)
```

为了方便调用，我们将上述两个函数集成到一个函数中，这里需要你完成这个函数的编写。

**Exercise**: 函数完成的是前向传播算法的*LINEAR->ACTIVATION* 模块。对应的数学公式是: $A^{[l]} = g(Z^{[l]}) = g(W^{[l]}A^{[l-1]} +b^{[l]})$。公式里的激励函数 "g" 可以是 sigmoid() 或 relu()。  



	# GRADED FUNCTION: linear_activation_forward
	
	def linear_activation_forward(A_prev, W, b, activation):
	    """
	    Implement the forward propagation for the LINEAR->ACTIVATION layer
	
	    Arguments:
	    A_prev -- activations from previous layer (or input data): (size of previous layer, number of examples)
	    W -- weights matrix: numpy array of shape (size of current layer, size of previous layer)
	    b -- bias vector, numpy array of shape (size of the current layer, 1)
	    activation -- the activation to be used in this layer, stored as a text string: "sigmoid" or "relu"
	
	    Returns:
	    A -- the output of the activation function, also called the post-activation value 
	    cache -- a python dictionary containing "linear_cache" and "activation_cache";
	             stored for computing the backward pass efficiently
	    """
	    
	    if activation == "sigmoid":
	        # Inputs: "A_prev, W, b". Outputs: "A, activation_cache".
	        ### START CODE HERE ### (≈ 2 lines of code)
	        Z, linear_cache = linear_forward(A_prev, W, b)
	        A, activation_cache =  sigmoid(Z)
	        ### END CODE HERE ###
	    
	    elif activation == "relu":
	        # Inputs: "A_prev, W, b". Outputs: "A, activation_cache".
	        ### START CODE HERE ### (≈ 2 lines of code)
	        Z, linear_cache = linear_forward(A_prev, W, b)
	        A, activation_cache = relu(Z)
	        ### END CODE HERE ###
	    
	    assert (A.shape == (W.shape[0], A_prev.shape[1]))
	    cache = (linear_cache, activation_cache)
	
	    return A, cache

-------------------------------------------------------

	A_prev, W, b = linear_activation_forward_test_case()
	
	A, linear_activation_cache = linear_activation_forward(A_prev, W, b, activation = "sigmoid")
	print("With sigmoid: A = " + str(A))
	
	A, linear_activation_cache = linear_activation_forward(A_prev, W, b, activation = "relu")
	print("With ReLU: A = " + str(A))

----------------------
**Expected output**:
       
<table style="width:35%">
  <tr>
    <td> With sigmoid: A  </td>
    <td > [[ 0.96890023  0.11013289]]</td> 
  </tr>
  <tr>
    <td> With ReLU: A  </td>
    <td > [[ 3.43896131  0.        ]]</td> 
  </tr>
</table> 

**Note**: In deep learning, the "[LINEAR->ACTIVATION]" computation is counted as a single layer in the neural network, not two layers.   
  

### d) L层模型  

为了方便L层模型的实现，我们需要一个函数，重复执行`linear_activation_forward`（RELU) **L-1** 次，最后执行一次 `linear_activation_forward`(SIGMOID)。   

   ![](/images/images_2018/model_architecture_kiank.png)    

<center> **Figure 2** : *[LINEAR -> RELU] $\times$ (L-1) -> LINEAR -> SIGMOID* model</center>

**Exercise**: 完成上述要求。  

**Instruction**: 变量 `AL` 表示 $A^{[L]} = \sigma(Z^{[L]}) = \sigma(W^{[L]} A^{[L-1]} + b^{[L]})$。(即 `Yhat`, i.e., $\hat{Y}$.)  

**Tips**:  

- 使用已经写好的函数    
- 使用loop循环执行 [LINEAR->RELU] (L-1) 次   
- 持续更新"caches" 列表。 `list`增加 `c` ，可以使用方法`list.append(c)`。

	# GRADED FUNCTION: L_model_forward
	
	def L_model_forward(X, parameters):
	    """
	    Implement forward propagation for the [LINEAR->RELU]*(L-1)->LINEAR->SIGMOID computation
	    
	    Arguments:
	    X -- data, numpy array of shape (input size, number of examples)
	    parameters -- output of initialize_parameters_deep()
	    
	    Returns:
	    AL -- last post-activation value
	    caches -- list of caches containing:
	                every cache of linear_activation_forward() (there are L-1 of them, indexed from 0 to L-1)
	    """
	
	    caches = []
	    A = X
	    L = len(parameters) // 2                  # number of layers in the neural network
	    
	    # Implement [LINEAR -> RELU]*(L-1). Add "cache" to the "caches" list.
	    for l in range(1, L):
	        A_prev = A 
	        ### START CODE HERE ### (≈ 2 lines of code)
	        A, cache = linear_activation_forward(A_prev, parameters["W"+str(l)], parameters["b"+str(l)], "relu")
	        caches.append(cache)
	        ### END CODE HERE ###
	    
	    # Implement LINEAR -> SIGMOID. Add "cache" to the "caches" list.
	    ### START CODE HERE ### (≈ 2 lines of code)
	    AL, cache = linear_activation_forward(A, parameters["W"+str(L)], parameters["b"+str(L)], "sigmoid")
	    caches.append(cache)
	    ### END CODE HERE ###
	    
	    assert(AL.shape == (1,X.shape[1]))
	            
	    return AL, caches

-------------------------------

	X, parameters = L_model_forward_test_case_2hidden()
	AL, caches = L_model_forward(X, parameters)
	print("AL = " + str(AL))
	print("Length of caches list = " + str(len(caches)))

---------------------------------
<table style="width:50%">
  <tr>
    <td> AL </td>
    <td > [[ 0.03921668  0.70498921  0.19734387  0.04728177]]</td> 
  </tr>
  <tr>
    <td> Length of caches list  </td>
    <td > 3 </td> 
  </tr>
</table>

现在，你已经完成了整个前向传播，输入是X，输出是行向量 $A^{[L]}$，其中包含你的预测值。 所有的中间变量记录在了 "caches"中。 利用 $A^{[L]}$， 你可以计算代价函数的值。



## 5 - 代价函数

为了检验你的模型是否有效，需要计算代价函数的值。  

**Exercise**:  

计算公式 $J$ 如下所示：   
 $$-\frac{1}{m} \sum\limits_{i = 1}^{m} (y^{(i)}\log\left(a^{[L] (i)}\right) + (1-y^{(i)})\log\left(1- a^{[L](i)}\right)) \tag{7}$$


	# GRADED FUNCTION: compute_cost
	
	def compute_cost(AL, Y):
	    """
	    Implement the cost function defined by equation (7).
	
	    Arguments:
	    AL -- probability vector corresponding to your label predictions, shape (1, number of examples)
	    Y -- true "label" vector (for example: containing 0 if non-cat, 1 if cat), shape (1, number of examples)
	
	    Returns:
	    cost -- cross-entropy cost
	    """
	    
	    m = Y.shape[1]
	
	    # Compute loss from aL and y.
	    ### START CODE HERE ### (≈ 1 lines of code)
	    cost = -1/m * np.sum(Y * np.log(AL) + (1-Y) * np.log(1-AL))
	    ### END CODE HERE ###
	    
	    cost = np.squeeze(cost)      # To make sure your cost's shape is what we expect (e.g. this turns [[17]] into 17).
	    assert(cost.shape == ())
	    
	    return cost


-------------------------------------------

	Y, AL = compute_cost_test_case()
	
	print("cost = " + str(compute_cost(AL, Y)))

---------------------------------------------

**Expected Output**:

<table>

    <tr>
    <td>**cost** </td>
    <td> 0.41493159961539694</td> 
    </tr>
</table>

