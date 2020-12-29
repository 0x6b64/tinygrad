<p align="center">
  <img src="https://raw.githubusercontent.com/geohot/tinygrad/master/docs/logo.png">
</p>

--------------------------------------------------------------------

![Unit Tests](https://github.com/geohot/tinygrad/workflows/Unit%20Tests/badge.svg)

For something in between a [pytorch](https://github.com/pytorch/pytorch) and a [karpathy/micrograd](https://github.com/karpathy/micrograd)

This may not be the best deep learning framework, but it is a deep learning framework.

The Tensor class is a wrapper around a numpy array, except it does Tensor things.

tinygrad is also a city in Russia.

### Installation

```bash
pip3 install git+https://github.com/geohot/tinygrad.git --upgrade
```

### Example

```python
from tinygrad.tensor import Tensor

x = Tensor.eye(3)
y = Tensor([[2.0,0,-2.0]])
z = y.matmul(x).sum()
z.backward()

print(x.grad)  # dz/dx
print(y.grad)  # dz/dy
```

### Same example in torch

```python
import torch

x = torch.eye(3, requires_grad=True)
y = torch.tensor([[2.0,0,-2.0]], requires_grad=True)
z = y.matmul(x).sum()
z.backward()

print(x.grad)  # dz/dx
print(y.grad)  # dz/dy
```

## Neural networks?

It turns out, a decent autograd tensor library is 90% of what you need for neural networks. Add an optimizer (SGD, RMSprop, and Adam implemented) from tinygrad.optim, write some boilerplate minibatching code, and you have all you need.

### Neural network example (from test/test_mnist.py)

```python
from tinygrad.tensor import Tensor
import tinygrad.optim as optim

class TinyBobNet:
  def __init__(self):
    self.l1 = Tensor.uniform(784, 128)
    self.l2 = Tensor.uniform(128, 10)

  def forward(self, x):
    return x.dot(self.l1).relu().dot(self.l2).logsoftmax()

model = TinyBobNet()
optim = optim.SGD([model.l1, model.l2], lr=0.001)

# ... and complete like pytorch, with (x,y) data

out = model.forward(x)
loss = out.mul(y).mean()
optim.zero_grad()
loss.backward()
optim.step()
```

## GPU and Accelerator Support

tinygrad supports GPUs through PyOpenCL.

```python
from tinygrad.tensor import Tensor
(Tensor.ones(4,4).gpu() + Tensor.ones(4,4).gpu()).cpu()
```

### ANE Support?!

If all you want to do is ReLU, you are in luck! You can do very fast ReLU (at least 30 MEGAReLUs/sec confirmed)

Requires your Python to be signed with `ane/lib/sign_python.sh` to add the `com.apple.ane.iokit-user-access` entitlement, which also requires `amfi_get_out_of_my_way=0x1` in your `boot-args`. Build the library with `ane/lib/build.sh`

```python
from tinygrad.tensor import Tensor

a = Tensor([-2,-1,0,1,2]).ane()
b = a.relu()
print(b.cpu())
```

Warning: do not rely on the ANE port. It segfaults sometimes. So if you were doing something important with tinygrad and wanted to use the ANE, you might have a bad time.

### Adding an accelerator

You need to support 15 basic ops:

```
Add, Sub, Mul, Pow              # binary ops (with broadcasting)
Relu, Log, Exp                  # unary ops
Sum, Max                        # reduce ops (with axis argument)
Dot, Conv2D                     # matrix multiplication and conv
Reshape, Transpose              # moving things around ops
Pad2D, Unpad2D                  # stupid slices
```

## ImageNet inference

Despite being tiny, tinygrad supports the full EfficientNet. Pass in a picture to discover what it is.

```bash
ipython3 examples/efficientnet.py https://upload.wikimedia.org/wikipedia/commons/4/41/Chicken.jpg
```

Or, if you have a webcam and cv2 installed

```bash
ipython3 examples/efficientnet.py webcam
```

PROTIP: Set "GPU=1" environment variable if you want this to go faster.

PROPROTIP: Set "DEBUG=1" environment variable if you want to see why it's slow.

### tinygrad also supports GANs

See `examples/mnist_gan.py`

<p align="center">
  <img src="https://raw.githubusercontent.com/geohot/tinygrad/master/docs/mnist_by_tinygrad.jpg">
</p>

## The promise of small

tinygrad will always be below 1000 lines. If it isn't, we will revert commits until tinygrad becomes smaller.

### Running tests

```bash
python3 -m pytest
```

### TODO

* Train an EfficientNet on ImageNet
* Add a language model. BERT?
* Add a detection model. EfficientDet?
* Reduce code
* Increase speed
* Add features

