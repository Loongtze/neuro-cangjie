# Deep Learning for Cangjie

## Backgrounds

[Cangjie](https://en.wikipedia.org/wiki/Cangjie_input_method) is a Chinese input method, based on the visual structure of the ideographs. It assigns 1-5 letters from a-z to various sub-components in the character, from left to right, top to down. 

18% of the characters have multiple Cangjie codes. This is because Unicode unifies minor variants of the 'same' ideographs, whose strokes and structures might differ slightly but are significant enough to be considered different in Cangjie codes.

Ideographs in Unicode is still in expansion. Most of the newly adopted ideographs don't have a definite pronunciation in many East Asian languages, phonetic inputs are not viable for them, leaving Cangjie one of the viable visually based input methods. Each time a new extension comes out, it is dreadful to encode those new ones with Cangjie codes. So this neural network aims at assigning Cangjie codes to new ideographs automatically.

## The Model

In this task, I adopt a revised encoder-decoder model with attention. The graph of the model is depicted below: (Note, all activations are ignored)

![Model Graph](/Figures/graph.svg "Model Graph")

4 models are used, an encoder which is a CNN network that converts a 64 × 64 × 1 bitmap into features. The other 3 models rely on the features generated by encoder. Among them are 2 dense layer based models that predict code length and potential multiple code representations count respectfully. These 2 models are trained independently from the encoder, since the features from the encoder is rich enough to include the necessary information to make such predictions. The outputs from these 2 models, together with  the features from encoder are fed into the decoder.

The decoder is a RNN, starting from an indication of 'start', it predict the next code using the current code ('start' at the beginning) and the above mentioned inputs, and then the prediction is fed beck as the current code to continue this process, until the maximum code length is reached. Codes shorter than the maximum length are padded with 'end' to the tail. The decoder will learn that 'end' is only followed with 'end', so no special treatment is needed for early stoping with shorter codes. The decoder and encoder are trained together.

There is an extra 5th model (not depicted in the figure above), which is a reduced decoder, used in the first step training. The whole training process is divided into 2 steps, during the first, the reduced decoder is used to focus the training of encoder. After 20 epochs, the full decoder starts to train, replacing the reduced one. This 2 steps training process increases the robustness and reduces the total training time.

## Training the Model

To train this model yourself, you need to download [Hanazono](https://fonts.jp/hanazono/) fonts, the 2 fonts (A and B) are the only fonts covering all Unicode ideographs. A copy of Cangjie code is already included here ([source](https://github.com/rime-aca/rime-cangjie6)).

The model is written with [TensorFlow](https://www.tensorflow.org) >= 2.0.0, other framework requirements are included in requirements.txt file. It is highly recommended to use a GPU to train this model. Training time with GPU is usually 3-4 hours, and can be longer than 2 days without GPU.

The training and validation result from the last run is shown in this figure below as the light blue and red lines. Notice there is a jump at 60 epochs, that's when teacher forcing was turned off. Dark blue and orange lines are from a previous run with teacher forcing always on. The loss on the right from different runs are not comparable, red line has a higher loss because teacher forcing on validation set is always off in the latest run.

<p align="middle">
  <img src="/Figures/accuracy.svg" alt="Accuracy" width="350"/>
  <img src="/Figures/loss.svg" alt="Loss" width="350"/>
</p>

In end of this latest run, accuracy on the training set reached 93%, on the validation set (which was not used in the training process in any form) reached 89% after 120 epochs. This is good enough to put into actual use.

Among those wrong predictions in validation, around 80% correspond to predicted probabilities of 90% or lower. Whereas among those correct predictions, only less than 20% correspond to predicted probabilities of 90% or lower. So, in addition to a predictions themselves, the predicted probabilities can be a good indicator of the correctness of predictions. This result is included in the notebook file.

## License

WTFPL
