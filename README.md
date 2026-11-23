# AC3 Meal Estimator

Unzip the package and open `index.html` in Chrome or Edge. The first AI run downloads the vision model and needs an internet connection. The download may take a few minutes. No API key is needed. The photo is processed in the browser. If the model cannot load, you can still select dishes and use the calculator.

1. Choose a photo of the main plate. Crop out the separate rice bowl, soup, drinks and the scale display.
2. Select **Suggest dishes**. Review the proposed dishes and correct the selection as needed.
3. Enter the scale reading in grams. For the plate photographed at 0.354 kg, enter 354. Use the food-only checkbox if the scale was tared or if you are explicitly trying that assumption.
4. Compare the two estimates. Open **Edit portion and calorie values** to replace the illustrative inputs with your own. Download the result as a CSV if needed.

## How it works

The app runs a quantized CLIP model (Xenova/clip-vit-base-patch32) through Transformers.js in the browser. It classifies the full photo and four overlapping regions against a fixed list of dish groups and excluded items. The app suggests a dish when a region's top label is a dish group, its relative score is at least 0.30, and it leads the next label by at least 0.08. These are prototype thresholds. The score is relative to the candidate list. The app produces dish suggestions for the plate as a whole.

Default calories are the sum of each selected dish's default grams × kcal per 100 g ÷ 100. With a food-only scale reading, the app multiplies all the default portions by the same factor so their sum equals the entered grams. Unknown dishes can count toward portion weight. Their calories are included only if a calorie value is supplied.

## Data and evaluation

`AC3_pilot_data.csv` records the group's reported weights and leaves the label and evaluation fields blank. Use the same plate ID for every photo of one serving. Record the dishes the AI suggested before any manual corrections. The six buffet-counter photos are outside the plate inventory.

The recorded weights are the values used for billing. Whether the scale subtracted the plate's weight is unverified for those samples. Rice costs HK$5 separately and is outside the scale weight, along with the separate soup and water. The 427.7 g value is based on the group's corrected record.

The default values in the prototype are for demonstration. They do not describe measured AC3 recipes. The scale adjustment also assumes that each dish keeps the same share of the plate. To compare calorie errors, set aside new plates with each component weighed separately and obtain an appropriate kcal/100 g reference for each component. Report the number of independent plates and the source of every calorie reference.

Model: https://huggingface.co/Xenova/clip-vit-base-patch32  
USDA FoodData Central: https://fdc.nal.usda.gov/  
Hong Kong Centre for Food Safety: https://www.cfs.gov.hk/english/nutrient/searchmenu.php
