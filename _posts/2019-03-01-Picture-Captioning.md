---
title: Picture Captioning
categories:
 - Python
 - Image Processing
 - Recognition
tags: Python Recognition
image: https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/img_3.png
---

In this post we introduced the implemention of a encode-decoder model for picture captioning. The training dataset we utilized is the coco dataset which contains both the image and its caption extracted by human. We also used resnet-152 to extract the features of images and embedding layers to lower the dimension of text space. A simple LSTM cell class is also implemented to be used in the text interpretation.

### Implementation of LSTM Cell

Detailed explanation of LSTM network can be seen here: https://colah.github.io/posts/2015-08-Understanding-LSTMs/. By referencing the LSTM structure from the post, the LSTM cell can be implemented as the following:

```java
class LSTMCell(nn.Module):
    """
    LSTM cell implementation
    Given an input x at time step t, and hidden and cell states: hidden = (h_(t-1), c_(t-1)),
    you will implement a LSTM unit to compute and return (h_t, c_t)
    hints: consider use linear layers to implement the several matrices W_* 
    Note: you just need to implement a one-layer LSTM unit
    """
    def __init__(self, input_size, hidden_size):
        super(LSTMCell, self).__init__()
        self.sigmoid = nn.Sigmoid()
        self.tanh = nn.Tanh()
        self.input_size = input_size + hidden_size
        self.wf = nn.Linear(self.input_size, hidden_size)
        self.wi = nn.Linear(self.input_size, hidden_size)
        self.wc = nn.Linear(self.input_size, hidden_size)
        self.wo = nn.Linear(self.input_size, hidden_size)
     

    def forward(self, x, hidden):
        h_0, c_0 = hidden
        features = torch.cat((h_0,x),1)
        ft = self.sigmoid(self.wf(features))
        it = self.sigmoid(self.wi(features))
        ct = self.tanh(self.wc(features))
        ot = self.sigmoid(self.wo(features))
        new_ct = ft*c_0 + it*ct
        ht = ot*self.tanh(new_ct)
        return (ht,ct)
```

Where `wf`, `wi`, `wc` and `wo` are the gate streams in the LSTM cell. A RNN with LSTM cells can thus be implemented on top of this:

```python
class LSTM(nn.Module):
    def __init__(self, input_size, hidden_size):
        super(LSTM, self).__init__()

        self.hidden_size = hidden_size # dimension of hidden states
        self.lstmcell = LSTMCell(input_size, hidden_size)

    def forward(self, x, states):
        h0, c0 = states
        outs = []
        cn = c0[0, :, :]
        hn = h0[0, :, :]
        for seq in range(x.size(1)):
            hn, cn = self.lstmcell(x[:, seq, :], (hn, cn))
            outs.append(hn)
        out = torch.stack(outs, 1)
        states = (hn.unsqueeze(0), cn.unsqueeze(0))
        return out, states
```

### Implementation of Encoder-Decoder Model

In our implementation of encoder we transformed the image into its feature domain with a pre-trained 152 layerred-resnet. 

```python
class Encoder(nn.Module):
    def __init__(self, embed_size):
        """Load the pretrained ResNet-152 and replace top fc layer."""
        super(Encoder, self).__init__()
        resnet = models.resnet152(pretrained=True)
        modules = list(resnet.children())[:-1]      # delete the last fc layer.
        self.resnet = nn.Sequential(*modules)
        self.linear = nn.Linear(resnet.fc.in_features, embed_size)
        self.bn = nn.BatchNorm1d(embed_size, momentum=0.01)
        
    def forward(self, images):
        """Extract feature vectors from input images."""
        with torch.no_grad():
            features = self.resnet(images)
        features = features.reshape(features.size(0), -1)
        features = self.bn(self.linear(features))
        return features
```

In the decoder part, we first get the embedded features by inputing captions into the embedding layer. Then we concatenated the embedded caption together with the extracted features of images. Then the concatennated features are fed into the LSTM and the output is compared to the initial captions in the dataset. The `forward` function of decoder is implemented as following:

```
def forward(self, features, captions, lengths):
        """Decode image feature vectors and generates captions."""
        
        # initialize the hidden state and cell state
        states = (autograd.Variable(torch.zeros(1, features.size(0), self.hidden_size).cuda()),
                autograd.Variable(torch.zeros(1, features.size(0), self.hidden_size).cuda()))
        
        # in the init function. There are four steps: (1) extract 
        # word embeddings from captions; (2) concatenate visual feature (features, bx1xd) and extracted
        # word embedding (bxtxd) and obtain the new features (bx(t+1)xd); (3) feed the new features into 
        # LSTM with the initialized states; (4) use a linear layer to project the feature to vocabulary 
        # space for training with a cross-entropy loss function.
        # captions: (128,31)
        word_embd = self.embed(captions)
        features = features.view(features.shape[0],1,features.shape[1])
        fw_cat = torch.cat((features,word_embd),1)
        lstm_out = self.lstm(fw_cat,states) 
        
        outputs = self.linear(lstm_out[0])   # outputs: (batch_size, t+1, vocab_size)

        outputs =  pack_padded_sequence(outputs, lengths, batch_first=True)
        return outputs[0]
```

### Results

We downloaded several images online and tested our model against Microsoft's captioning robot. The images are from different categories so we can test the model on a broader perspective.

The first image is a sandwich cut in a half with cheese and vegetables:

![]()

The Microsoft's robot gives the following result:

```
I think it's a sandwich cut in half.
```

Meanwhile our model outputs the result of:

```
<start> a sandwich with a slice of cheese and cheese . <end>
```

The second image is an image of a baseball player:

![]()

Microsoft's robot recognizes the action correctly:

```
I think it's a baseball player holding a bat on a field.
```

However, our model is over interpreting what the player is doing:

```
<start> a baseball player is swinging a bat at a ball <end>
```

The third is a picture of natural landscape:

![]()

Microsoft's result is:

```
I think it's a snow covered mountain.
```

But our model is still over interpreting:

```
<start> a person is skiing down a hill in the snow . <end>
```

Obviously the Microsoft's CaptionBot is more precise. Maybe it cannot capture all the details but it will not provide inaccurate information. On the other hand, our own's caption robot is roughly correct about the object in the images, like it can tell about the topic is a sandwich or a mountain. However, sometimes our own's robot cannot provide precise description as well as providing some totally wrong information. For instance, it will tell there is a person skiing in the third image but we cannot observe this by our own eyes.

The problem of inaccurate details may come from lack of more complex structures of our model thus cannot capture features in smaller scale. Or only training it by one epoch will make it less accurate. Maybe we can ﬁx this issue some how by enlarging the training dataset, training the more longer or changing to another architecture.

There are several ways to improve the captioning model: 1. More training and ﬁne tuning of the models, like the image model can be trained ﬁrst for several epochs and then joint tuning both image and language models. 2. Model ensembling: ensemble multiple image models to extract more and precise features for training the language model. 3. Use beam search in language modeling: keep the top k sequences ever generated so far. We can also search for several diﬀerent k values to achieve the highest CIDER value.







