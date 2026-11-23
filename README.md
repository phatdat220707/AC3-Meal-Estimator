# AC3 Meal Estimator

Open `index.html` in Chrome or Edge. The first AI run downloads the model and needs internet access. Photos stay in the browser, and you can select dishes by hand if the model can't load.

1. Add a photo of the main plate and click **Suggest dishes**. If the photo includes a scale, rice bowl or a lot of background, use **Crop to plate** and try again.
2. Check the suggestions against the plate. The default estimate uses sample portions for the dishes you select.
3. If you have a scale reading, enter it in grams. Choose **Food only (tared)** if the scale excluded the plate. Choose **Food and plate** if you weighed both, then enter the empty plate's weight. Use **Not sure** if you don't know how it was weighed.
4. Enter the weight of each known portion in **Enter dish g**. The app shows a subtotal and, when it has a food-only weight, the grams still unassigned. It shows a whole-plate estimate once all selected dishes have a weight and calorie value, the portions match the food weight within 1 g, and you confirm that the list covers the plate.
5. To change the sample values, open **Edit portion and calorie values**. Use **Download results CSV** to save the suggestions, your choices and the weighing information.

## How it works

```text
Plate photo → five image views → CLIP ranks labels for each view
                                    ↓
                         JavaScript filters and combines suggestions
                                    ↓
                     Selected dishes + portion weights → calorie estimate
```

### Where the model runs

The interface, image handling, and calorie calculation are all in `index.html`. On the first click of **Suggest dishes**, `loadModel()` imports Transformers.js 3.7.2 from jsDelivr. The library downloads the pretrained `Xenova/clip-vit-base-patch32` model files from Hugging Face and runs the 8-bit quantized ONNX model using WebAssembly in the browser. The photo stays in the browser throughout this process. [Model-loading code](index.html#L353) · [Model files](https://huggingface.co/Xenova/clip-vit-base-patch32/tree/main/onnx)

CLIP is a model trained to match images with text. It converts the image and each candidate description into numeric representations, then scores how well they match. For example, the app asks how well a view matches “a photo of cooked fish fillet pieces” and compares that score with the scores for the other descriptions. This is called *zero-shot image classification* because the app supplies the candidate descriptions at prediction time and uses a model that was already trained. [OpenAI's CLIP overview](https://openai.com/index/clip/)

### How one photo can produce several suggestions

1. The browser reads the photo into a preview. When you click **Suggest dishes**, `views()` uses Canvas to read pixels from five views of the active image: the whole image and four overlapping corner crops. If you used **Crop to plate**, all five views come from that trimmed image. The corner crops use fixed coordinates, so a view can contain parts of several dishes. [Image preparation code](index.html#L376)
2. For each view, CLIP ranks five dish descriptions (cooked chicken, cooked fish, fried tofu, leafy greens, and glass noodles) and four descriptions that can rule out a view (plain rice in a bowl, soup in a bowl, an empty plate or tabletop, and mixed food with no clear dish). [Candidate labels](index.html#L134)
3. The app keeps **at most one dish per view**. The highest-ranked label must be one of the five dishes, score at least `0.30`, and lead the runner-up by at least `0.08`. The scores depend on which labels are in the candidate list. A dish that ranks second in a view is skipped for that view, even if its score is high. [Classification and filtering code](index.html#L405)
4. The app combines the accepted dishes, removes duplicates, and displays them under **AI suggestions**. If you have yet to change the dish selection, the app also checks the suggested dishes for you. [Result display code](index.html#L419)

For example, on a plate with fish, greens, and tofu, fish might rank first in both the full image and the top-left crop. Greens might win in the top-right crop. If tofu leads in another crop with `0.39` against fish at `0.35`, the `0.04` gap is too small and that crop adds nothing. If the last crop also suggests greens, the result is **Cooked fish, Leafy greens**. You can add the tofu by hand. The result can contain several dishes, just one, or none. Since the model ranks labels for fixed views, it can miss dishes that share a view with a more visually prominent dish. It provides no dish locations or portion weights.

### How the app calculates calories

The calorie calculation uses the dishes currently selected on the page. For each dish, it multiplies grams by kcal per 100 g and divides by 100. The app shows an estimate based on the default sample portions and a separate subtotal for portions whose grams you enter. For example, at `150 kcal/100 g`, the default `80 g` of fish gives `120 kcal`, while an entered portion of `100 g` gives `150 kcal` in the subtotal. [Calculation code](index.html#L147)

The scale reading checks how much of the food weight the entered portions account for. A dish with a known weight and no calorie value can count toward the weight while its calories stay outside the subtotal. The app labels the subtotal **Whole-plate estimate** once every selected dish has both a weight and a calorie value, the entered weights match the food-only weight within 1 g, and you confirm that the selected dishes cover the plate. [Code that reads the inputs and updates the estimate](index.html#L215)

## Try the 354 g plate

In the pilot, the original photo returned no suggestions. A tighter crop suggested cooked fish in the CPU test. Set the weighing mode to **Not sure** unless you have verified that the scale excluded the plate. Selecting fish alone gives a default estimate of 120 kcal for 80 g at a sample density of 150 kcal/100 g.

To test the weight check, assume the 354 g reading excludes the plate and choose **Food only (tared)**. With no dish weights entered, the app shows 354 g unassigned and leaves the subtotal blank. Enter 80 g for fish and the subtotal becomes 120 kcal, with 274 g still unassigned. The 80 g is a test input. The earlier prototype assigned all 354 g to fish in this case. The current version has you enter the weight for each dish.

## Pilot data

`AC3_pilot_data.csv` lists the reported weights and leaves space for dish labels and evaluation. Use the same plate ID for different views of one serving. The canteen used these readings for billing, and we could not verify that it subtracted the plate's weight. The scale weight excludes the separately served rice, soup and water. The 427.7 g entry follows the group's corrected record.

The sample calorie values do not describe measured AC3 recipes. To compare estimation error, weigh each food component and use a documented kcal/100 g value for the closest matching preparation.

Model: https://huggingface.co/Xenova/clip-vit-base-patch32  
Reference databases: https://fdc.nal.usda.gov/ and https://www.cfs.gov.hk/english/nutrient/searchmenu.php
