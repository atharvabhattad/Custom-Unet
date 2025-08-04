# Polygon Colorization Project Report

## Introduction

This project details the development of a convolutional neural network designed to color a polygon given its grayscale outline and a corresponding color. The primary goal was to create a unet model capable of accurately predicting the color of the polygon, a task that presented unique challenges related to loss function design and model capacity.

## Hyperparameters

The development process involved an iterative approach to hyperparameter tuning and model design. The initial attempts with a simple model and standard settings resulted in a typical failure mode. The final, successful configuration was achieved through a series of strategic changes.

## What We Tried

**Initial Model**: A simple UNet-like architecture with a default learning rate (1e−3) and a standard Mean Squared Error (MSE) loss.

**Initial Failure Mode**: The model failed to learn the task, consistently outputting a completely white image. This occurred because the large white background of the target images allowed the model to achieve a low MSE by simply predicting white everywhere, a "lazy" local minimum.

## Final Settings and Rationale

The final, high-performing model was a result of the following changes:

- **Learning Rate (LR)**: Reduced to 5e−4. A lower learning rate was crucial to prevent the model from overshooting the optimal solution and settling in a trivial local minimum.

- **Loss Function**: A custom pixel-wise weighted MSE loss was implemented. This was the most critical fix. By giving pixels within the polygon outline a higher weight (foreground_weight = 5.0), the model was forced to prioritize learning the correct color for the polygon itself, rather than the background.

- **Training Epochs**: The training duration was significantly increased. WandB reports were generated for runs of 100, 500, and 1000 epochs, demonstrating that longer training times were necessary for the model to converge to a perfect solution.

## Architecture

The final architecture is a deeper, more robust UNet that effectively leverages skip connections to maintain spatial information while processing the input.

- **UNet Design**: The model is based on a standard UNet encoder-decoder structure. The encoder consists of four downsampling blocks, and the decoder uses four corresponding upsampling blocks with bilinear upsampling. This deeper structure provides the necessary capacity to learn the intricate mapping from a grayscale outline and a color to a detailed RGB image.

- **Conditioning Method**: Color information is introduced into the network through a unique conditioning mechanism.

  - The target color is provided as a one-hot encoded vector.
  - This vector is passed through a two-layer linear embedding to create a rich feature vector.
  - This feature vector is reshaped into a feature map with the same spatial dimensions and channel depth as the UNet's deepest bottleneck layer.
  - This color feature map is then added element-wise to the bottleneck features. This method, often called Feature-wise Linear Modulation (FiLM), is an effective way to inject conditional information without altering the network's channel dimensions, allowing the color to modulate the learned deep features.

## Training Dynamics

The training process was closely monitored using Weights & Biases (WandB), which provided critical insights into the model's behavior.

- **Failure Mode and Fixes**: The initial "all white prediction" was the primary failure. The introduction of the weighted loss function successfully addressed this by providing a much stronger gradient signal for errors on the foreground pixels.

- **Qualitative Output Trends**: The WandB reports for the final model show a clear progression:
  - **Early Epochs (100)**: The model begins to move away from the pure white output. A faint, blurry, and often mixed color starts to appear within the polygon outline.
  - **Mid-Epochs (500)**: The predicted color becomes more accurate, and the edges of the colored polygon become sharper, but some noise or texture are still present in edges and corner.
  - **Late Epochs (1000)**: The model converges. The output becomes a near perfect, solid-colored polygon with clean edges that precisely match the input outline and the target color. The loss curves near flatten out, indicating that the model has reached a stable minimum.

## Key Learnings

- **Loss Function is King**: The choice of loss function is paramount, especially in tasks with a severe class imbalance. A simple, off-the-shelf loss function may not be sufficient, and a carefully designed custom loss is often required to guide the model's learning towards the desired outcome.

- **The Power of Experiment Tracking**: Tools like WandB are indispensable. They provided a clear visual record of the model's failure and subsequent recovery, allowing for rapid debugging and informed decision-making regarding hyperparameter adjustments.

- **Intelligent Conditioning**: Simply concatenating conditional information may not be the most effective strategy. Injecting a modulated feature map (as was done with the color embedding) into a deep bottleneck layer is a powerful technique for guiding a generative model's output.

## References
https://github.com/milesial/Pytorch-UNet
