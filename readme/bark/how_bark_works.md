# Note
I'm still unsure how bark works. I am trying to figure out how it works though. My current knowledge goes here.


## Different models
Bark has 3 different models, used for each step in the generation process. Every model also has a smaller version for quicker and less costly generation.
1. Semantics
   * i: A piece of text
   * p: Generate semantic tokens for this text
   * o: A list of semantic tokens
2. Coarse
   * i: A list of semantic tokens
   * p: Transform this list into audio
   * o: A numpy array with audio
3. Fine
   * i: A numpy array with audio
   * p: Reduce noise in the audio
   * o: A numpy array of audio

## Current voice cloning methods
### The process
1. Take an audio file and a transcription
2. Prepare the audio to be in the right format
3. Store how long audio takes
4. Generate semantic tokens from input text, with max_gen_duration_s as audio duration
5. Save to a numpy zip (.npz):

|  FileName | Content (.npy)                 |
|----------:|:-------------------------------|
| Semantics | the generated semantics        |
|      Fine | the audio file                 |
|    Coarse | first 2 rows of the audio file |

### Why is it so bad?
As you can see, on step 4, it generates semantic tokens. It does this by generating a random speaker saying the semantics. The problem with that is that these can be completely different from your actual semantics. Therefore, the voice cloning will only be successful if you spoke just like the AI did with the semantic tokens.

Basically, the method generates semantic tokens which say what you said, and combines it with the fine and coarse prompt created from your voice audio.

The problem is that your semantic tokens and fine and coarse prompts usually won't match. Because they are from two separate things. The semantic tokens are based on your input text, which could have a lot of variety. But your fine and coarse prompts are actually based on your audio.

## The next step
How would I improve the creation of speaker files for voice cloning?

As of my current knowledge, I don't think there's anything wrong with the creation of the fine and coarse prompts. As the actual audio is put into them. Just as it should be done.

Where the actual problem lies is in the semantic token creation. As I explained before, the tokens don't match your audio clip, as they are generated from your transcription, not your audio.

My current guess for fixing it has a few method proposals:
1. Find how the semantics are generated in bark. (This might not be publicly available)
   * This method would be very effective, and have a low time cost. But the question would be if it's possible with the data i've got.
2. Train your own neural network to convert audio files to semantic tokens
   * Creating the training data would be really easy, but also very time consuming, as there needs to be a lot of training data for it to understand all the tokens. I propose using a markov chain to create a lot of different prompts. The words may be nonsense, but can be used to train properly.
     1. Generate a bunch of different audios in bark
     2. Store the semantic tokens, and also store the output audio (fine generation)
   * Training the model (I could likely finetune a speech recognition model)
     1. Use the audio as the input, and the semantic tokens as the output
     2. Train on the audio-token-pairs until the model is able to accurately determine semantic tokens from an audio file