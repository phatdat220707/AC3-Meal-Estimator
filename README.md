# AC3 Meal Estimator

Open `index.html` in Chrome or Edge. The first AI run downloads the model and needs internet access. Photos stay in the browser, and you can select dishes by hand if the model can't load.

1. Add a photo of the main plate and click **Suggest dishes**. If the photo includes a scale, rice bowl or a lot of background, use **Crop to plate** and try again.
2. Check the suggestions against the plate. The default estimate uses sample portions for the dishes you select.
3. If you have a scale reading, enter it in grams. Choose **Food only (tared)** if the scale excluded the plate. Choose **Food and plate** if you weighed both, then enter the empty plate's weight. Use **Not sure** if you don't know how it was weighed.
4. Enter the weight of each known portion in **Enter dish g**. The app shows a subtotal and, when it has a food-only weight, the grams still unassigned. It shows a whole-plate estimate once all selected dishes have a weight and calorie value, the portions match the food weight within 1 g, and you confirm that the list covers the plate.
5. To change the sample values, open **Edit portion and calorie values**. Use **Download results CSV** to save the suggestions, your choices and the weighing information.

## How it works

The app uses quantized CLIP (Xenova/clip-vit-base-patch32) through Transformers.js. It scores the active photo and four overlapping crops. It keeps an in-scope dish if the dish ranks first in a view, scores at least 0.30 and leads the next label by at least 0.08. The scores depend on the candidate list.

To estimate a dish's calories, the app multiplies its grams by its kcal per 100 g and divides by 100. The scale reading checks how much of the plate the entered portions account for. A dish with no calorie value can count toward the weight while its calories stay outside the subtotal.

## Try the 354 g plate

In the pilot, the original photo returned no suggestions. A tighter crop suggested cooked fish in the CPU test. Set the weighing mode to **Not sure** unless you have verified that the scale excluded the plate. Selecting fish alone gives a default estimate of 120 kcal for 80 g at a sample density of 150 kcal/100 g.

To test the weight check, assume the 354 g reading excludes the plate and choose **Food only (tared)**. With no dish weights entered, the app shows 354 g unassigned and leaves the subtotal blank. Enter 80 g for fish and the subtotal becomes 120 kcal, with 274 g still unassigned. The 80 g is a test input. The earlier prototype assigned all 354 g to fish in this case. The current version has you enter the weight for each dish.

## Pilot data

`AC3_pilot_data.csv` lists the reported weights and leaves space for dish labels and evaluation. Use the same plate ID for different views of one serving. The canteen used these readings for billing, and we could not verify that it subtracted the plate's weight. The scale weight excludes the separately served rice, soup and water. The 427.7 g entry follows the group's corrected record.

The sample calorie values do not describe measured AC3 recipes. To compare estimation error, weigh each food component and use a documented kcal/100 g value for the closest matching preparation.

Model: https://huggingface.co/Xenova/clip-vit-base-patch32  
Reference databases: https://fdc.nal.usda.gov/ and https://www.cfs.gov.hk/english/nutrient/searchmenu.php
