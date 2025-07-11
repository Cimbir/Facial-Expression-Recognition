# Facial-Expression-Recognition

ამ პროექტის მიზანია ნეირონული ქსელების გამოყენებით ადამიანის სახეების დასწავლა და მათგან ემოციების ამოცნობა. სულ არის 7 ემოცია (0=Angry, 1=Disgust, 2=Fear, 3=Happy, 4=Sad, 5=Surprise, 6=Neutral), ხოლო ფასდება იმითი თუ რამდენად მაღალი accuracy დადო მოდელმა

> **გასათვალისწინებელი:**  
> აქ ძირითადად მიწერის პრეოქტისა და ექსპერიმენტების მიმდინარეობა. თვითონ მონაცემების განხილვა და უკეთესი ანალიზი მაქვს დაწერილი [wandb-ს რეპორტში](https://wandb.ai/dachis-none/emotion-recognition/reports/Emotion-Recognition-Report--VmlldzoxMzEzNzgwNw)

## Repository Structure

* `nn.py` - მთავარი სამუშაო ნოუთბუქი (ბევრი აბსტრაქციის გამო იქამდე მივედი, რომ უფრო კომფორტული იყო CNN-ებისა და FCNN-ების ერთად დაწერა)

* `README.md` - პროექტის აღწერა

## Preprocessing

მონაცემები შემოდის 48x48-ზომის სურათებად Grayscale ფორმატში. თავდაპირველად სურათების დამუშავებისას მხოლოდ ტენზორად გადამყავდა სურათი. შემდეგ, რაც უფრო მეტ არქიტექტურას ვტესტავდი, სხვადასხვა preprocessing მეთოდები გამოვიყენე:

* `ნორმალიზაცია` - ყველა სურათს გამოვაკელი სატრენინგო სურათების საშუალო მნიშვნელობა ტენზორად გადაყვანის შემდეგ, რამაც იგივე არქიტექტურის მქონე მოდელზე დაახლოებით 5%-იანი ზრდა აჩვენა ვალიდაციის ტესტზე. ამისი მიზეზი არის იგივე რაც ყველა ნორმალიზაციის შემთხვევაში - ნორმალიზებული მონაცემები უფრო სწრაფად ითვისება და უფრო სტაბილური ხდება ტრენინგის პროცესში(3-Layer-Down-NN-adam vs 3-1024-Layer-Down-NN)

* `შეტრიალება` - RandomHorizontalFlip()-ს გამოყენებით სურათების ნაწილს ჰორიზონტალურად ვაბრუნებდი, რომ არსებული სურათები ერთმანეთისგან რაღაც ნიშნით მაინც ცოტათი განსხვავდებოდნენ. ეს ძირითადად ტრენინგის ბოლოს დავამატე, როდესაც მინდოდა შემემცირებინა overfitting რაც ბოლო მოდელებში ხშირად იყო. ამ მიდგომის გამოყენებით ვალიდაცია მცირედით გაუმჯობესდა, მაგრამ ის gap რაც აქამდე ჰქონდა მოდელს, მკვეთრად შემცირდა: ~13% -> 2% (3-1-Up-Maxpool-Dropout-Layernorm-CNN-image-flip vs 3-1-Up-Maxpool-Dropout-Layernorm-CNN)

სხვა შესაძლო ცვლილებები შესაძლოა ყოფილიყო სურათის მცირედი დატრიალება, თუმცა დატაზე ყველა სურათი თითქმის იგივე ორიენტაციით იყო, რის გამოც აღარ ჩავთვალე ამ მოდიფიკაციის დამატება საჭირო

## Training

თავიდანვე ცხადი იყო რომ რადგანაც სურათები შემოდი input-ად, საჭირო იყო cnn-ები, თუმცა სანამ დავიწყებდი მათ გამოყენებას, თავიდან ვეცადე შედარებით მარტივი ნეირობული ქსელების სტრუქტურით ამეწყო სხვადასხვა მოდელები და მაგითიც დავკვირვებულიყავი სხვადასხვა მახასიათებლის მოდიფიკაციას. სატესტოდ ძირითადად ყველა მოდელს 10 ეპოქაზე ვტესტავდი, რათა თავიდანვე სწრაფად მენახა მოდელი თუ რამდენად კარგად მუშაობდა. ამის გარდაც, სანამ მოდელის ტრენინგზე გადავიდოდი, არქიტექტურაში გრადიენტების მოძრაობას ვაკვიდრებოდი მოდელის ოვერფიტინგით: თუ მოდელის train loss მალევე 0-დებოდა, ხშირად ეგ მოდელი ჩვეულებრივ ტრენინგზეც მალევე overfit-დებოდა, თუ ბოლოსაც ძლივს ჩადიოდა 0-მდე, ანუ გრადიენტები კარგად ვერ მოძრაობენ და ახალი ნაწილები უნდა დამემატებინა მოდელში რათა გრადიენტებს უკეთ ემოძრავათ ხოლო თუ 40-60 ეპოქაზე უკვე 0-ზე ჩადის, ეგ მოდელები შედარებით კარგად სწავლობენ და კარგ შედეგებს დებდნენ ხოლმე. მთლიანი დატა გავყავი შემდეგნაირად:

* 80% - ტრენინგი
* 10% - ვალიდაცია
* 10% - ტესტირება

### Training Info

#### Charts

* `Loss Progress` - ყოველი ეპოქისთვის loss ფუქნციის მნიშვნელობა

* `Accuracy Progress` - ყოველი ეპოქისას accuracy-ს მნიშვნელობა

* `Gradient-to-Weight Ratio over time` - რა %-ით იცვლება გრადიენტი წონებთან შედარებით

#### Metrics

* `weight_norm` - წონების სიდიდე(L2) ყოველ ეპოქაზე

* `train/val_loss/acc` - ტრენინგის/ვალიდაციის loss ფუქნციის მნიშვნეობა/accuracy-ს სიდიდე

* `loss/accruracy` - ტესტირების შედეგები

* `grad_ratio` - გრადიენტი პროცენტულად რამხელაა წონებზე

* `epoch_time` - დრო თითო ეპოქისთვის (train->val)

#### Media

* `Overfit Training Curve` - თავდაპირველი overfit-ის გრაფიკული ინფორმაცია

* `Training Curve` - ტრენინგის გრაფიკული ინფორმაცია

### Process

1. ვცვლიდი ქსელის სიღრმეს: 1-2-3 ფენები თითო 1024 ზომის(1-Layer-NN vs 2-1024-Layer-NN vs 3-1024-Layer-NN). აქ აღმოვაჩინე რომ 3 ფენის მქონე ქსელს ყველაზე ცუდი მაჩვენებელი ჰქონდა ვიდრე 1-ს ან 2-ს. დიდი ალბათობით ეს იმისი გამოწვეულია, რომ მოდელი კარგად ვერ ახერხებდა გრადიენტის გადაცემას, რადგან wandb-ზე გრადიენტებისა და პარამეტრების შემოწმებით 1-სა და 2-ს შედარებით უფრო გაშლილი იყო, ვიდრე 3, თუმცა რათქმაუნდა არცერთი არ კარგი

2. გრადიენტების გადაცემის პრობლემის მოსაგვარებლად თავიდან გამოვიყენე BatchNorm, თუმცა ამ და მომავალ run-ებშიც ერთი და იგივე შედეგი მეორდებოდა: მოდელი ყოველთვის overfit-დებოდა. ბეჩის ზომის გაზრდისა და რეგულარიზაციის სხვადასხვა ხერხის მიუხედავად, ვალიდაცია ყოველთვის მკვეთრად ჩამორჩებოდა ტრენინგს. ამას თავდაპირველი ოვერფიტინგის შემთხვევებიც კარგად აფიქსირებს ხოლმე. რაც შეეხება LayerNorm-ს, დიდად არ ცვლიდა შედეგს

3. ვცადე შრეების სხვადასხვა წყობები: 1 ზომის შრეები(3-1024-Layer-NN-128-Batch), შრეების ზომების შემცირება(3-1024-Layer-Down-NN) და შრეების ზომის გაზრდა და მერე შემცირება(3-1024-Layer-Hourglass-NN). ყველა გასინჯვისას თითქმის ერთნაირი შედეგი იდებოდა, თუმცა ონლაინ წყაროებიდან და უფრო მცირე სტრუქტურის გამო ვარჩიე შრეების შემცირების მიდგომა მომავალი მოდელებისთვის

4. dropout-ც ვცადე, მაგრამ ჯერ-ჯერობით არანაირი თვალსაჩინო შედეგი არ მოუცია, რადგან ისედაც მარტივი მოდელი იყო და რეგულარიზაცია მაინცდამაინც არ სჭირდებოდა. თუმცა, როგორც მოსალოდნელი იყო, მეტი dropout-თ მოდელი უარეს accuracy-ს დებს. ამი მერე dropout მოვდე BatchNorm-თან ერთად. Dropout დაეხმარა. რაც უფრო დიდი იყო, მით უფრო შემცირდა სხვაობა ტრენინგსა და ვალიდაციას შორის და ამასთან ერთად ზოგადად ვალიდაციის მაჩვენებელიც გაიზარდა(3-1024-Layer-Down-Batchnorm-NN vs 
3-Layer-Down-Dropout-0.1-Batchnorm-NN vs 3-Layer-Down-Dropout-0.5-Batchnorm-NN)

5. უფრო ღრმა მოდელებზე გადავედი, მაგრამ დიდად გაუმჯობესება არ მოუცია (3-1024-Layer-Down-NN vs 4-Layer-Down-NN vs 5-Layer-Down-NN vs 6-Layer-Down-NN)

6. ვცადე სხვადასხვა ოპტიმიზატორები: SGD, Adam, RMSprop. SGD-მა ყველაზე ცუდი შედეგი აჩვენა, ხოლო Adam-მა და RMSprop-მა თითქმის ერთნაირი შედეგი დადეს, თუმცა RMSprop-ს უფრო მეტი overfitting ჰქონდა. საბოლოოდ, Adam-ი გამოვიყენე. ამასთან ერთად, ამ მოდელებზე დავამატე სურათების თავდაპირველი ნორმალიზაცია, რამაც კი გააუმჯობესა შედეგები, მაგრამ ამავდროულად შეეტყო overfitting-ც. ამის გამო შემდეგ დავამატე l2 რეგულარიზაცია, რომელმაც მკვეთრად შეამცირა ეს სხვაობა. learning rate-ს შეცვლაც ცვადე, მაგრამ მაინცდამაინც დიდი ცვლილება არ მოუცია. აქედან გამომდინარე მომავალ მოდელებს გავაგრძელებდი adam-თ განახლებას, l2-თ რეგულარიზაციასა და სურათების თავდაპირველ ნორმალიზაციას

7. ბოლოს უბრალოდ წინა ყველა დანამატი ავიღე და სხვადასხვა კომბინაციებით სხვადასხვა კარგი მოდელი დავასწავლე, რომელთაგან საუკეთესო აღმოჩნდა 
3-Layer-Down-Dropout-0.3-Batchnorm-NN-adam-l2

8. ახლა გადავედი უკვე CNN-ებზე. თავიდან შედარებით მარტივად დავიწყე: ვცადე 1 და 3 რაოდენობის შრის კონვოლუცია. ორივე შემთხვევაში დიდი სხვაობა იყო ტრენინგსა და ვალიდაციას შორის, მაგრამ ზოგადად 3-შრიანმა გაცილებით უკეთესი შედეგი დადო თავისთავად. შემდეგ დავამატე Maxpool და შრეების რაოდენობაც გავზარდე, მაგრამ თავდაპირველი overfit-სას ჩანდა უკვე, რომ მოდელში გრადიენტებს უჭირდათ მოძრაობა

9. ამ პრობლემის გამოსასწორებლად თავიდან მივუბრუნდი BatchNorm-ს, მაგრამ როგორი დამატებითი მიდგომაც არ ვარჩიე: dropout, l2 reg, შრეების რაოდენობის შემცირება, ყოველთვის დიდი სხვაობა იყო ტრენინგსა და ვალიდაციას შორის, ამიტომაც BatchNorm-ს მაგივრად გადავედი Layernorm-ზე, რომელმაც გრადიენტების პრობლემა გამოასწორა overfitting-ის პრობლემის გარეშე

10. აქვე დავამატე სურათების შეტრიალებაც, რომლის შედეგებიც უკვე ზევით დავწერე

11. მომავალ და საბოლოო მოდელებში წინად არსებულ მოდიფიკაციებს ვაერთიანებდი და ვნახულობდი შედეგებს. ღრმა ქსელებში უკვე დიდი პრობლემა ხდებოდა გრადიენტების მოძრაობა layernorm-ის მიუხედავად. ამის გამოსასწორებლად მოდელებს თავდათან ვუმატებ Batch ნორმალიზაციას კონვოლუციურ შრეებზე, რაც ზოგადად აუმჯობესებს გრადიენტების მოძრაობას, ამიტომ ყველა მოდელში ვტოვებ. ამავდროულად ვუმატებ LeakyReLU-სა და Residual COnnection-ებს, მაგრამ მოდელებში დიდად შედეგებს არ ცვლიან. ამათგან საბოლოოდ ხელით გადავარჩიე საუკეტესო არქიტექტურა

12. ბოლოს გავუშვი ჰიპერპარამეტრების sweep, რათა აღმომეჩინა საუკეთესო მოდელი. ამის მიუხედავად, ხელით ამორჩეული მაინც აღმოჩნდა საუკეთესო სხვა ყველასთან შედარებით, ამიტომაც საბოლოოდ საუკეთესოდ ეგ მოდელი დავტოვე

### Best Model

საბოლოოდ, საუკეთესო მოდელი გამოდგა 3-2-Up-Maxpool-Dropout-Layernorm-CBatchnorm-CNN არქიტექტურის 3-2-Best run-დან. ბოლო მოდელებიდან, რომლებიც საუკეთესო შედეგის დასადებად იყო შექმნილი, ერთადერთი ეს იყო, რომელმაც მიაღწია 0.6 accuracy-ს და overfitting-იც არ განუცდია. wandb-ს რეპორტში კარგად ჩანს თუ რამდენად უსწრებს სხვა დანარჩენ მოდელებს

## Testing

### Testing Info

#### Chart

* `Confusion Matrix Curve` - ტესტირების შედეგად დადებული შედეგების confusion matrix

#### Metrics

* `loss/accruracy` - ტესტირების შედეგები

* `test/(recall/precision/f1)_x` - recall, precision და f1 მნიშვნელობები მე-x ემოციისთვის ტესტირებისას