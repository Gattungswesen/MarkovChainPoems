# MarkovChainPoems
# Generated or Not Generated: That is the Question

In the case of a Markov chain, a square stochastic matrix represents the probability that each state will transition to each other state. In such a matrix, the probability of going from state n to state m is given by the entry at row n, and column m in the matrix. As a result, the sum of all entries along a row must be 1, since the sum of the probabilities of all possible next words must end up being 1.

In our case of generating poems, the probabilities in the matrix are computed from the frequency of occurrence of each string of words in the training text being followed by each possible next word. If we were working with individual letters (rather than whole words) this can be observed with vowels. The frequency for instance, of the characters &#39;e&#39; or &#39;c&#39; occurring next to the letter &#39;t&#39;, is far more likely that &#39;e&#39; follows the letter &#39;t&#39; than any consonant. Higher frequencies of sequential occurrence increase the odds of that occurrence appearing in the generated text.

If the probability that word 2 follows word 1 is given by P (word2 | word1), and we assume that this probability is roughly equal for all pairs of words, then the probability that 3 words happen sequentially in the sample text can be approximated by

P (word2 | word1) P (word3 | word2) ⋍ P(word2 | word1)^2

This implies that as we increase the number of words being considered as part of the current state of the system, the size of the required sample text to have samples of each possible state used in multiple contexts (with different words following) scales exponentially [1]. As a result, we would need exponentially increasing amounts of training data to adequately train the model with more words in the current state.

**Implementation**

In order to use Markov chains to generate poetry, we assign each unique word or sequence of words in our training data set a number, representing the state of the system. If the last word generated translates to a number x, then we randomly select the next word using the probabilities stored in row x of the matrix. Better predictions can be generated if we consider the current state to be the last 2 words, rather than the last 1 word of generated text.

In our code, this is implemented by generating the row/column numbers from the string of characters containing both words (separated by a space), rather than a string containing just the last word. This can be further extended by using arbitrarily many words as keys for the matrix. The limiting factor is that if we consider more than roughly 5 words at a time, it becomes very likely that those 5 consecutive words only occurred once in the entirety of the training data. This is a problem because when we try to generate the next word in the poem, the only possible next word is the one used in the original poem. It&#39;s very likely that this pattern continues, and that the generated poem ends up being exactly the same as the original, which obviously defeats the whole purpose.

The solution to this problem would be to have more sample text so that strings of many (\&gt;5) words at a time occur in many different contexts within the text. Unfortunately, the required text length increases nonlinearly.

A natural limitation of this method in the real world is that if you have Markov chains that consider the state of the poem to depend on only the last &#39;n&#39; words, the Markov chain will not create structures longer than &#39; n+1&#39; words, so the poem will make some sense locally, but looking at scales much larger than n+1 words it will be nonsense. In this poem, any two words may seem linked, but upon reading the whole poem it would be meaningless. The solution to this is to increase the number of words being considered as part of the state of the system. This also has a problem though: when there are too many consecutive words being considered (we found the number is about 5 words at a time for our poetry data) it is statistically unlikely that the sequence of 5 words has been written in multiple poems/contexts in our training text. With only one option, there is a 100% chance of being the next word, leading to the generated poems being exact copies of one existing poem.

The workaround to this is more data, with enough more poems there will be more options for the nth word, but the amount of data (in words) needed to give other options to the last word grows exponentially. Balancing this tradeoff was a major part of this project, as we are trying to make the best poems possible without copying. Since there aren&#39;t infinitely large databases of poetry, we generated poetry by using only 2 words at a time to represent the state of the system. See Appendix B for examples of sample text generated using multiple-state string lengths.

Our code runs in two phases: *learning*, and *generation*.
