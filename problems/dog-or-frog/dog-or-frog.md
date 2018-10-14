# Dog or Frog

> Dressing up dogs are kinda the new thing, see if you can get this lovely girl ready for her costume party. Dog Or Frog

> Hints:  
> This really is a ML problem, read the hints in the problem for more details..

Let's take a look at the `solution_template.py` on the website:

```python
from keras.applications.mobilenet import preprocess_input
from keras.models import load_model
from keras.preprocessing.image import img_to_array, array_to_img
from PIL import Image
from imagehash import phash


IMAGE_DIMS = (224, 224)
TREE_FROG_IDX = 31
TREE_FROG_STR = "tree_frog"


# I'm pretty sure I borrowed this function from somewhere, but cannot remember
# the source to cite them properly.
def hash_hamming_distance(h1, h2):
    s1 = str(h1)
    s2 = str(h2)
    return sum(map(lambda x: 0 if x[0] == x[1] else 1, zip(s1, s2)))


def is_similar_img(path1, path2):
    image1 = Image.open(path1)
    image2 = Image.open(path2)

    dist = hash_hamming_distance(phash(image1), phash(image2))
    return dist <= 1


def prepare_image(image, target=IMAGE_DIMS):
    # if the image mode is not RGB, convert it
    if image.mode != "RGB":
        image = image.convert("RGB")

    # resize the input image and preprocess it
    image = image.resize(target)
    image = img_to_array(image)
    image = np.expand_dims(image, axis=0)
    image = preprocess_input(image)
    # return the processed image
    return image


def create_img(img_path, img_res_path, model_path, target_str, target_idx, des_conf=0.95):
    test = Image.open(img_path).resize(IMAGE_DIMS)
    test = prepare_image(test)
    model = load_model(model_path)

    # TODO: YOUR SOLUTION HERE


    test = test.reshape((224,224,3))
    img = array_to_img(test)
    img.save(img_res_path)


if __name__ == "__main__":
    create_img("./trixi.png", "./trixi_frog.png", "./model.h5", TREE_FROG_STR, TREE_FROG_IDX)
    assert is_similar_img("./trixi.png", "./trixi_frog.png")
```
We see that the index of tree frog as the resultant vector of the model is `31`. The model is loaded from `model.h5` (also included on the website), which is used on the website on an uploaded image, and then outputted.

The goal of this problem is simply to construct an adversarial example. An adversarial example is simply a visually similar image that causes a neural network to output a completely different result. We want to start with `trixi.png`, output `trixi_frog.png` from this solution script such that is is fairly similar to the original, but the neural network is tricked into thinking it is a tree frog.

After doing some research, we find the library [CleverHans](https://github.com/tensorflow/cleverhans). Looking at the [documentation for attacks](https://cleverhans.readthedocs.io/en/latest/source/attacks.html), we see many different attacks to construct adversarial examples. Many of these will likely work, but we had success with the momentum iterative method. Here's our script that uses this attack:

```python
from keras.applications.mobilenet import preprocess_input
from keras.models import load_model
from keras.preprocessing.image import img_to_array, array_to_img
from keras import backend as K
from PIL import Image
from imagehash import phash
import numpy as np
import tensorflow as tf
import cleverhans.attacks


IMAGE_DIMS = (224, 224)
TREE_FROG_IDX = 31
TREE_FROG_STR = "tree_frog"

# ...

def create_img(img_path, img_res_path, model_path, target_str, target_idx, des_conf=0.95):
    print "Loading..."
    tval = TREE_FROG_IDX
    target = np.zeros((1,1000))
    target[0][tval] = 1
    print target # target is just TREE_FROG_IDX one-hot encoded
    test = Image.open(img_path).resize(IMAGE_DIMS)
    test = prepare_image(test)
    model = load_model(model_path)

    sess = K.get_session()
    print "Calculating initial probability of tree_frog: "
    print sess.run(model(tf.constant(test))[0,tval])

    print "Beginning attack..."
    method = cleverhans.attacks.MomentumIterativeMethod(model, sess=sess)
    
    # overwrite the image with the output of the attack
    # the values of eps, eps_iter, and nb_iter were adjusted so the final probability > 0.95 but the image isn't too different
    test = method.generate_np(test, eps=0.3, eps_iter=0.06, nb_iter=10, y_target=target)

    print "Attack done, final probability of tree_frog:"
    print sess.run(model(tf.constant(test))[0,tval])

    test = test.reshape((224,224,3))
    img = array_to_img(test)
    img.save(img_res_path)


if __name__ == "__main__":
    create_img("./trixi.png", "./trixi_frog.png", "./model.h5", TREE_FROG_STR, TREE_FROG_IDX)
    assert is_similar_img("./trixi.png", "./trixi_frog.png")
```
The outputted `trixi_frog.png` can be uploaded to the website, and will then give us the flag.