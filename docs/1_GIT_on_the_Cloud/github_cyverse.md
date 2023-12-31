# Connecting Cyverse to GitHub

## Log in to Cyverse

1. Go to the Cyverse user account website [https://user.cyverse.org/](https://user.cyverse.org/)

<img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/61b8c22a-bed3-457a-b603-736fd8e59568">

2. Click `Sign up` (if you do not already have an account)

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/73dc39a4-30f2-4017-8f0d-1006db24d25b">

3. Head over to the Cyverse Discovery Environment [https://de.cyverse.org](https://de.cyverse.org), and log in with your new account.

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/41970a8d-c434-4075-9dd4-fbcd0f2ea07c">

   You should now see the Discovery Environment:

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/0dcd0048-a4e3-469c-bd28-5a5574c5dec3">

4. We will give you permissions to access the Hackathon app. If you haven't already, let us know that you need access

## Open up an analysis with the hackathon environment (Jupyter Lab)

1. From the Cyverse Discovery Environment, click on `Apps` in the left menu

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/4bb45960-0a9b-4e3d-af83-fd424dae9bf4">

2. Select `JupyterLab Earthlab`

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/31bdaedf-efa7-4778-bd3b-b790efa128b9">

3. Configure and launch your analysis - the defaults are fine for now:

   <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/1108f008-3c4d-4216-80f6-fa6f9a63248f">

  <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/b82f08d7-4d21-4fba-9e40-f0a4ac492898">

  <img width="881" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/ecb6051c-17bc-4e79-a40a-04d5949ed472">


4. Click `Go to analysis`:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/f9ea6ffe-cfd7-44c3-9ca2-cb90740df6a4">

5. Now you should see Jupyter Lab!
   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/b7702aee-b561-440d-92a9-daa9990a7f96">

## Set up your GitHub credentials

1. From Jupyter Lab, click on the GitHub icon on the left menu:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/8c694ad7-2454-4ffc-b422-e95382d0205f">

2. Click `Clone a Repository`:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/42f691b0-efbd-45f8-8554-157c70902abe">

3. Paste the link to the innovation-summit-utils [https://github.com/CU-ESIIL/innovation-summit-utils.git](https://github.com/CU-ESIIL/innovation-summit-utils.git) and click `Clone`:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/50c07e00-fb17-48a5-a460-ab5fc730384c">


4. You should now see the `innovation-summit-utils` folder in your directory tree (provided you haven't changed directories from the default `/home/jovyan/data-store`

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/7e455efc-7d56-47d4-a854-563c38a14acc">

5. Go into the `innovation-summit-utils` folder:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/bbc0955b-008a-4376-9d4e-9644982ccb8c">

6. open up the `configure_github_ssh.ipynb` notebook by double-clicking:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/a3cbf78c-b1ce-448b-b02b-a648fd074136">

7. Select the default `earth-analytics-python` kernel

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/7a59013b-160d-4a91-81d8-e64f77acbbfe">

8. Now you should see the notebook open. Click the `play` button at the top. You will be prompted to enter your GitHub username and email:

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/50f617c6-8b77-4e32-99cd-d2dcdb7dc5dc">

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/46115471-60de-4fa9-918b-9a3cbf2d28af">

   <img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/8a71d74b-0e53-417a-9a30-ac31915c8ce4">

9. You should now see your Public Key. Copy the WHOLE LINE including `ssh-ed25519` at the beginning and the `jovyan@...` at the end
<img width="1004" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/876c3294-34e9-4d05-86e2-046a69b6d843">

10. Go to your GitHub settings page (you may need to log in to GitHub first):

    <img width="1462" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/9dc79edc-e527-4b98-a94f-4d500571b97a">

11. Select `SSH and GPG keys`

    <img width="1462" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/427be66e-b1e2-46ca-93b6-e3585a7c7fb3">

12. Select `New SSH key`

    <img width="1462" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/a33ad793-ea4d-4d44-9a8f-bde1b04f69c0">

13. Give your key a descriptive name, paste your ENTIRE public key in the `Key` input box, and click `Add SSH Key`. You may need to re-authenticate with your password or two-factor authentication.:

    <img width="1462" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/74888d16-5042-4f3b-abaa-34498c83a276">

14. You should now see your new SSH key in your `Authentication Keys` list! Now you will be able to clone private repositories and push changes to GitHub from your Cyverse analysis!

    <img width="1462" alt="image" src="https://github.com/CU-ESIIL/hackathon2023_datacube/assets/3465768/a4bdfada-f7f4-40b7-a1af-41b18d7bd3e6">

> NOTE! Your GitHub authentication is ONLY for the analysis you're working with right now. You will be able to use it as long as you want there, but once you start a new analysis you will need to go through this process again. Feel free to delete keys from old analyses that have been shut down.
