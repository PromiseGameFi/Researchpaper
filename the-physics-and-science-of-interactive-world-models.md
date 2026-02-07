---
title: "The Physics and Science of  Interactive World Models"
llm_description: "*."
---





# The Physics and Science of  Interactive World Models

## Abstract



***

## 1. Introduction

Interactive digital worlds have long relied on human-defined rules. Traditional game engines calculate the position of objects using discrete equations for motion and force. This method requires manual programming for every physical interaction.

Neural world models change this approach. Systems like Google DeepMind’s Genie 3 and Tencent’s Hunyuan treat the physical world as a sequence of visual patterns. By training on millions of hours of video, these models learn to predict the most likely next frame in a sequence. This prediction includes the visual appearance of the world and its physical behavior. This paper details the mathematical structures that allow these models to create stable, interactive, and physically consistent environments.

***

## 2. Mathematical Framework

The science of interactive world models relies on three distinct mathematical stages.

### A. Spatio-Temporal Tokenization

The model must compress high-resolution video into a manageable data format. It uses a Video Autoencoder, typically a VQ-VAE (Vector Quantized Variational Autoencoder), combined with Spatio-Temporal Transformers.

* The Math: An input video V is mapped to a latent space z. The encoder produces continuous vectors that are quantized into discrete tokens from a codebook C:

<div align="center"><img src="https://latex.codecogs.com/svg.image?z_q%20=%20\text{argmin}_{e_j%20\in%20C}%20||E(V)%20-%20e_j||^2" alt=""></div>

* The Science: By encoding both space and time together, the tokens capture both the content of the frame and the direction of movement. This is the foundation of the model's physical memory.

### B. Latent Action Modeling (LAM)

Interactive worlds require user control. However, most training videos do not have labels for keyboard or controller inputs. The LAM learns to infer these actions by analyzing the difference between consecutive frames.

<div align="center"><img src="https://latex.codecogs.com/svg.image?p(a_t%20|%20x_{t},%20x_{t+1})" alt=""></div>

* The Science: This creates a mathematical vocabulary of actions. The model learns that certain pixel changes correspond to concepts like "move forward" or "jump" without being told those words.

### C. Detailed Autoregressive Dynamics (The Physics of Probability)

The physics of an interactive world is defined as a transition density. The model learns the joint distribution of a sequence where S is state and a is action:

<div align="center"><img src="https://latex.codecogs.com/svg.image?P(S_{1:T},%20a_{1:T})%20=%20\prod_{t=1}^{T}%20P(S_t%20|%20S_{%3Ct},%20a_{1:t})" alt=""></div>

The Proof of Emergent Gravity: In a traditional engine, the position of an object is calculated using a formula. In a neural world model, the Transformer maximizes the likelihood of the token s\_t that represents an object at a lower position based on training data. The probability mass shifts toward lower positions:

<div align="center"><img src="https://latex.codecogs.com/svg.image?P(S_t%20=%20\text{lower\_pos}%20|%20S_{%3Ct})%20%3E%20P(S_t%20=%20\text{higher\_pos}%20|%20S_{%3Ct})" alt=""></div>

### D. The Loss Function (ELBO)

To ensure the latent space z represents the physical world accurately, the model is trained using the Evidence Lower Bound (ELBO).

<div align="center"><img src="https://latex.codecogs.com/svg.image?\mathcal{L}%20=%20\mathbb{E}_{q(z|x)}[\log%20p(x|z)]%20-%20\beta%20D_{KL}(q(z|x)%20||%20p(z))" alt=""></div>

The Reconstruction Term ensures the generated world looks like the real video. The KL Divergence forces the model to keep the latent space organized, allowing for smooth transitions when a user provides an action input.

***

## 3. Emergent Physics and Consistency

The most significant aspect of these models is Emergent Physics. The model does not contain a formula for gravity. Instead, gravity emerges as a statistical necessity based on the training data.

### Temporal Consistency

To maintain a world for several minutes, models like Genie 3 use a state buffer. This allows the AI to reference frames from earlier in the session. This prevents the "drifting" common in early video models where walls or floors would change shape when the user looked away.

### Geometry Injection

Hunyuan World Voyager improves on pixel prediction by using RGB-D (Red, Green, Blue, and Depth) joint generation. By understanding the 3D distance of every pixel, the model ensures that objects do not pass through each other. This creates a grounded sense of solid objects.

***

## 4. Hardware and Real-Time Inference

For these mathematical models to work in a playable way, they must achieve low latency.

* KV Caching: To avoid recalculating the entire history at every frame, the model stores previous mathematical values in memory. This reduces the computational complexity from O(T^2) to O(T).
* Quantization: The model weights are compressed from 32-bit decimals to 8-bit integers. This allows the physics calculations to run on standard graphics cards at 24 to 30 frames per second.

***

## 5. Method

Building these models involves a hierarchical training pipeline:

{% stepper %}
{% step %}
### Phase 1: Train the VQ-VAE

Train the VQ-VAE on diverse video datasets to create a universal visual vocabulary.
{% endstep %}

{% step %}
### Phase 2: Train the Latent Action Model

Train the Latent Action Model to identify controllable elements within those videos.
{% endstep %}

{% step %}
### Phase 3: Co-train the Dynamics Transformer

Co-train the Dynamics Transformer to predict future tokens based on actions.
{% endstep %}

{% step %}
### Inference

The model takes a text prompt to start the first frame. It then enters a loop where user inputs are mapped to the latent action space and the Transformer generates the next frame in real-time.
{% endstep %}
{% endstepper %}

***

## 6. Conclusion

The science of AI-driven world models move physics from the realm of logic to the realm of probability. Genie 3 and Hunyuan demonstrate that high-parameter Transformers can simulate complex environments that are playable and consistent. As these models scale, the need for traditional physics code will decrease. This allows for interactive digital realities generated entirely through mathematical inference.

***

## 7. References

* Google DeepMind (2025). _Genie 3: A New Frontier for World Models._
* Tencent Hunyuan Team (2025). _HunyuanWorld 1.0: Generating Immersive 3D Worlds._
* Bruce et al. (2024). _Genie: Generative Interactive Environments._
* Vaswani et al. (2017). _Attention is All You Need._
