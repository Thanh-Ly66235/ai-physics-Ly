# ai-physics-Ly

## What this notebook does

`week02_Ly.ipynb` is the Week 2 mini-assignment for the course. It builds a minimal, reproducible Python environment for querying a large language model programmatically, defines a `query_llm(prompt, model)` wrapper around the **Anthropic** Python SDK, and then interrogates the model (`claude-haiku-4-5-20251001`) with physics questions whose answers
can be independently verified. The notebook records each model response verbatim, states the accepted value from an authoritative source (PDG, NIST, or a standard textbook), computes or states the discrepancy, and classifies the response as **correct**, **approximately correct**, or **hallucinated**. At least one hallucination or hard limit is identified and discussed with physical reasoning, and at least one correct response is validated against a known source.

## Reproducing the results from scratch

1. **Clone the repository**

I, first, created the repository using Github (with README initialized). I locally created a file on Syzygy, and then I changed the directory on the terminal of Syzygy:
   ```bash
   cd "PHYS 7440 Gen AI/Assignment 1/ai-physics-Ly"
   ```
The gitignore is created by using:
   ```bash
   echo -e ".env\n__pycache__/\n*.pyc" > .gitignore
   ``` 
Then I committed and then pushed to git:
 ```bash
   git add .gitignore
   git commit -m "Add .gitignore to block credentials and cache files"
   git push origin main
   ``` 

2. **Create the environment** (conda is recommended; the notebook was developed on Python 3.13 in the `base` conda environment described by `environment.yml`):
   ```bash
   conda create -n ai-physics python=3.11
   conda activate ai-physics  
   ``` 

Then install the following:
   ```bash
   pip install openai>=1.0 
   pip install anthropic 
   pip install litellm
   pip install python-dotenv
   pip install tiktoken
   ``` 
Note: Since I'm installing on Syzygy, both numpy and matplotlib do not need to be installed (as they already have installed). 

The enviornment is then exported and then committed with:
   ```bash
   conda env export > environment.yml
   git add environment.yml
   git commit -m "Add environment.yml configuration file"
   ``` 
Next,  Create a file named `.env` in the repository root containing the Anthropic API key:
   ```bash
   echo 'ANTHROPIC_API_KEY=key' > .env
   ```

Then I created a ipynb file at the same directory, and work on it. The notebook contains the code that has the function as described previously.

## Reflection on LLM reliability in a physics contex

Working through this assignment made concrete a lesson that is easy to state and hard to internalize: an LLM's fluency is uncorrelated with its correctness, and its step-by-step reasoning is no guarantee of soundness. The model handled canonical, time-stable quantities well as it computed the Schwarzschild radius of a 10 M black hole correctly (≈ 29.5 km, matching the textbook value from r_s = 2GM/c²) and gave a competent qualitative description of the immersed boundary method in the context of cell blebbing. But when asked to calculate the dimensionless gravitational-wave strain amplitude h for a binary black hole merger with a 30 M chirp mass at 400 Mpc, the model produced a response that is a textbook example of a pure mathematical hallucination. Three distinct failures compound in that single answer. First, the formula itself is wrong: the model wrote h = (4/c⁴)(G³/d_L)M_c⁵(πf_GW)^{10/3}, which is dimensionally inconsistent, whereas the correct standard quadrupole formula is h = (4/d_L)(GM_c/c²)^{5/3}(πf_GW/c)^{2/3}. Second, the arithmetic is fabricated even on its own terms: the final division step shows (1.21734 × 10¹⁵⁷)/(9.93154 × 10⁵⁸), which should yield a result on the order of 10⁹⁸, yet the model reports h = 1.226 × 10⁻²¹. Third, and most revealing, the model landed on a "plausible answer" magnet: 10⁻²¹ is exactly the strain magnitude ubiquitously quoted in LIGO press releases and introductory articles about GW150914. The model's token-prediction attention heads were pulled toward that statistical anchor, and it back-fitted a fake formula and fake arithmetic to reach the answer it already "knew" it wanted. This is the most dangerous failure mode in AI-assisted physics: not a wildly wrong answer, but a confidently structured, step-by-step, plausibly-magnituded answer that passes a casual sanity check while being dimensionally, arithmetically, and physically wrong at every stage. The practical takeaway is that LLM outputs must be treated as hypotheses to be checked, never as sources. Step-by-step reasoning does not guarantee correctness (it can be as fabricated as the final answer). Every formula must be verified dimensionally, every numerical claim must be traced to a primary reference, and any quantity that is frontier, recent, or statistically framed should be assumed unreliable until independently confirmed. The environment built in this assignment, with secrets managed properly, and dependencies pinned, which exists precisely so that every number entering a calculation has a traceable provenance.