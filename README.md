# AC3 Meal Estimator

Open `index.html` in Chrome or Edge. The first AI run downloads the vision model and needs an internet connection. No API key is needed. The photo is processed in the browser. If the model cannot load, you can still select dishes and use the calculator.

1. Choose a photo and select **Suggest dishes**. Open **Crop to plate** to trim the edges, then run suggestions again.
2. Review the selected dishes. The default estimate covers only selected dishes and uses illustrative portion sizes.
3. Optionally enter the scale reading in grams. Choose **Food only (tared)** if the scale excluded the plate. For **Food and plate**, enter the empty plate weight. Leave **Not sure** when tare is unknown.
4. Enter each dish's weight under **Enter dish g**. The app shows the subtotal for the portions entered and flags unassigned food weight. It shows a whole-plate estimate only when the user confirms all foods are represented, all dishes have a weight and energy density, and the portions match the food weight within 1 g.
5. Open **Edit portion and calorie values** to change the illustrative inputs. Use **Download results CSV** to keep the AI suggestions, user selections, and weighing inputs together.

## How it works

The app runs a quantized CLIP model (Xenova/clip-vit-base-patch32) through Transformers.js. It scores the active photo and four overlapping crops against dish groups and excluded items. A dish is suggested when a region's top label is a dish group, its relative score is at least 0.30 and it leads the next label by at least 0.08. Scores are relative to the candidate list.

For each selected dish, the app multiplies its portion in grams by its kcal per 100 g and divides by 100. The plate reading checks how much of the food mass is accounted for. A dish without a calorie density can count toward the food weight, while its energy remains outside the subtotal.

## 354 g demonstration

The original photo returned no suggestions. A tighter view in the pilot CPU run suggested cooked fish. Keep the weighing mode as **Not sure** for the recorded 354 g unless tare is confirmed. If fish is the sole selected category, the default estimate is 120 kcal from 80 g at an illustrative 150 kcal/100 g.

To test the allocation check under an explicit food-only assumption, enter 354 g and select **Food only (tared)**. With no dish weights entered, the app should show 354 g unassigned and no weight-based calorie subtotal. If you enter 80 g for fish as a test input, the subtotal is 120 kcal and 274 g remains unassigned. The 80 g input here is an example, not a component measurement. Earlier pilot results used the original proportional calculator, which has been replaced.

## Data and evaluation

`AC3_pilot_data.csv` contains the group's reported weights and blank fields for labels and evaluation. Use the same plate ID for every view of the same serving. The recorded weights are billing readings, and whether the scale subtracted the plate is unverified for these samples. Rice is sold separately and is outside the scale weight, along with the separate soup and water. The 427.7 g value follows the group's corrected record.

The calorie density and portion defaults are illustrative. To compare estimation error, weigh individual food components and use a documented kcal/100 g reference matched to preparation.

Model: https://huggingface.co/Xenova/clip-vit-base-patch32  
Reference databases: https://fdc.nal.usda.gov/ and https://www.cfs.gov.hk/english/nutrient/searchmenu.php
