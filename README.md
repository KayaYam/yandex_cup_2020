# Разбор решения 

### Строганов Глеб, 4-ое место в финале

> Весь текст написан с целью помочь участникам будущих чемпионатов чуть лучше к ним подготовиться. На момент участия я только заканчивал тренинг на Яндекс.Практикуме и знаний в DataScience и програмировании в общем у меня было чуть больше чем 0. Поэтому прошу не критиковать особо за кривость кода. Сам сейчас уже вижу что многие вещи можно (и нужно!) было сделать по другому. 

> Так как это было первое (и пока единственное) для меня соревнование то сам я шел на него вооруженный одним единственным советом, найденным в сети:

>> Как можно быстрее создать хоть какой то бэйзлайн и потом уже его улучшать. 

> Поэтому свой "гайд" писал из того, что мне самому было бы интересно и полезно узнать перед стартном чемпионата. Так как я понятия не имею какой уровень подготовки у вас на момент прочтения, то часть текста можно отнести совсем в категорию "ликбез". Такой текст буду выделять ***курсивом***

_____________________________________________________________________

> Финал проходил в интерактивном режиме. Если вы читаете этот текст в рамках подготовки к соревнованиям от Яндекс то вот вам первая рекомендация: Ознакомьтесь с системой интерактивного формата заранее.

> Для меня эта система была совсем новой, поэтому часть времени ушло на то чтобы понять как это все работает. По счастью организаторы приложили полностью рабочий вариант решения обеспечивающее минимальный скор, но позволяющее разобраться в механике. Часть этого варианта перекочевала в мое финальное решение без изменений. В частности функция получения данных

    def read_train_data(target_count, class_count, budget):
        print('req 0 0 5500') 
        sys.stdout.flush()
        train_count = int(input())
        train_data_targets = []
        train_data_categories = []
        train_data_vectors = []
    
        for _ in range(train_count):
            input_line = input().strip().split()
            train_data_targets.append(list(map(int, input_line[:target_count])))
            train_data_categories.append(list(map(int, input_line[target_count:target_count + class_count])))
            train_data_vectors.append(list(map(float, input_line[target_count+class_count:])))
     
        return train_data_targets, train_data_categories, train_data_vectors

> Очевидное решение которое бы реализовал сейчас, но на которое не хватило времени и соображаловки во время чемпионата - парсить input сразу в датафрейм pandas. 

> Другая опция встречающаяся в финальных решениях других участников - переписать эту функцию так чтобы можно было ее использовать и для первого запроса и для дозапросов (о них ниже) То что я это не сделал можно объснить только "измененным состоянием сознания" в котором я пребывал все эти двое суток ;) 


> ***Напомню: таргетом у нас были три класса в трех бинарных столбцах (по результатам работы функции выше они попали в train_data_targets). Обучающие данные состояли из четырех категориальных признаков (train_data_categories) и 40 вещественных (train_data_vectors)***

> Процесс поиска решений трудно описать словами. Это чистое творчество при котором вертишь данные так и сяк, пока мозг (а точнее Дефолт Система Мозга) не выдаст какое то "озарение". Здесь таким озарением стало то, что при разных значениях в одном из четырех категориальных признаков резко увеличивается содержание одного из таргетов. Таким образом появился способ резко увеличить количество полезных записей - в запросе "по умолчанию" довольно большой процент записей не относился ни к одному из классов. 

> Следующая функция является ключевой во всем решении. Изначально планировал перед публикацией поправить нэйминг переменных, убрать повторяющиеся части в циклы, и т.д. Но потом... В конце концов именно таким я этот код отпавлял на проверку в систему и именно такой он и занял 4-ое место. 

    def get_ratio(field, cat_range, df):    
        top_th_1_cat_value = 0 
        top_th_1_rat_ratio = 0
        top_th_2_cat_value = 0
        top_th_2_rat_ratio = 0
        top_th_3_cat_value = 0 
        top_th_3_rat_ratio = 0 
        or_th1_ratio = df[df['theme_1'] ==1].shape[0] / df.shape[0]
        or_th2_ratio = df[df['theme_2'] ==1].shape[0] / df.shape[0]
        or_th3_ratio = df[df['theme_3'] ==1].shape[0] / df.shape[0]

        for val in range(cat_range):
            query = '(' + field + '==' + str(val) +')'
            temp_df = df.query(query)

            try:
                new_th1_ratio = temp_df[temp_df['theme_1'] == 1].shape[0] / temp_df.shape[0]
            except:
                new_th1_ratio = 0
            try:
                new_th2_ratio = temp_df[temp_df['theme_2'] == 1].shape[0] / temp_df.shape[0]
            except:
                new_th2_ratio = 0 
            try:
                new_th3_ratio = temp_df[temp_df['theme_3'] == 1].shape[0] / temp_df.shape[0]
            except:
                new_th3_ratio = 0 
            if new_th1_ratio > or_th1_ratio:
                if (new_th1_ratio / or_th1_ratio) > top_th_1_rat_ratio:               
                    top_th_1_cat_value = val 
                    top_th_1_rat_ratio = new_th1_ratio / or_th1_ratio

            if new_th2_ratio > or_th2_ratio: 
                if new_th2_ratio / or_th2_ratio > top_th_2_rat_ratio:
                    top_th_2_cat_value = val
                    top_th_2_rat_ratio = new_th2_ratio / or_th2_ratio

            if new_th3_ratio > or_th3_ratio:
                if new_th3_ratio / or_th3_ratio > top_th_3_rat_ratio:
                    top_th_3_cat_value = val 
                    top_th_3_rat_ratio = new_th3_ratio / or_th3_ratio

        return  top_th_1_cat_value, top_th_1_rat_ratio, top_th_2_cat_value, top_th_2_rat_ratio, top_th_3_cat_value, top_th_3_rat_ratio

> Что, уже думаете как бы это "развидеть"? )) 

> Да, изящества мало, но свою задачу эта функция выполнила - а именно позволила выявить три топовые категории, которые максимально содержат один из трех классов. Теперь дело оставалось за малым - собрать обучающую выборку:

- запрашиваем выборку по умолчанию (5500 записей);
- по ней определяем топ-3 категорий;
- дозапрашиваем по 1500 записей каждой из этих категорий.


    def main():
        cats=[]
        for c in range(40):
            cats.append('fe_' + str(c+1))
        columns = ['theme_1', 'theme_2','theme_3','cat_1','cat_2','cat_3','cat_4'] + cats
        test_columns = ['cat_1','cat_2','cat_3','cat_4'] + cats

        target_count, class_count, enbedding_dim = map(int, input().strip().split()) # T, C, F
        class_counts = list(map(int, input().strip().split())) # C_i
        budget = int(input()) # B

        train_data_targets, train_data_categories, train_data_vectors = read_train_data(target_count, class_count, budget)
        df_data = np.concatenate((train_data_targets, train_data_categories, train_data_vectors), axis = 1)

        df = pd.DataFrame(data = df_data, columns = columns)

        categories = ['cat_1', 'cat_2', 'cat_3', 'cat_4']
        ranges = [14,25,71,4]
        results = []
        for i in range(len(categories)):
            result = get_ratio(categories[i], ranges[i], df)
            results.append(result)

        themes_top =[0,0,0,0,0,0]

        for k in range(3):
            top_current_theme = 0
            for i in range(len(categories)):
                if results[i][k*2+1] > top_current_theme:
                    top_current_theme = results[i][k*2+1]
                    themes_top[k*2] = i
                    themes_top[k*2 + 1] = results[i][k*2]

        for k in range(3):
            current_cat_order = themes_top[k*2]
            current_cat_value = themes_top[k*2 + 1]
            current_cat_name = categories[current_cat_order]
            query = 'req ' + str(current_cat_order +1) + ' ' + str(current_cat_value) + ' 1500'

            print(query) 
            sys.stdout.flush()
            train_count = int(input())
            train_data_targets = []
            train_data_categories = []
            train_data_vectors = []

            for _ in range(train_count):
                input_line = input().strip().split()
                train_data_targets.append(list(map(int, input_line[:target_count])))
                train_data_categories.append(list(map(int, input_line[target_count:target_count + class_count])))
                train_data_vectors.append(list(map(float, input_line[target_count+class_count:])))
            train_data_targets, train_data_categories, train_data_vectors

            temp_df_data = np.concatenate((train_data_targets, train_data_categories, train_data_vectors), axis = 1)
            temp_df = pd.DataFrame(data = temp_df_data, columns = columns)
            df = pd.concat([df,temp_df])
            
> Собственно дальше все более менее просто:

- запрашиваем тестовые данные;
- парсим их в датафрейм;
- обучаем модели на обучающей выборке (у меня это было три Catboost'а, по одной на каждый класс. В решениях других участников видел и аналогичный подход и одну модель на все три класса);
- делаем predict
       

        print('test')
        sys.stdout.flush()

        test_count = int(input())
        test_data_categories = []
        test_data_vectors = []

        for _ in range(test_count):
            input_line = input().strip().split()
            test_data_categories.append(list(map(int, input_line[:class_count])))
            test_data_vectors.append(list(map(float, input_line[class_count:])))

        test_df_data = np.concatenate((test_data_categories, test_data_vectors), axis = 1)
        test_df = pd.DataFrame(data = test_df_data, columns = test_columns)


        train_features = df.drop(['theme_1', 'theme_2', 'theme_3'], axis = 1)

        params = {'loss_function':'Logloss', 
              'verbose': 0,
              'random_seed': 12345,
              'iterations': 100,
              'learning_rate': 0.305
             }

        cbc_1 = CatBoostClassifier(**params)
        train_target = df['theme_1']
        cbc_1.fit(train_features, train_target)

        cbc_2 = CatBoostClassifier(**params)
        train_target = df['theme_2']
        cbc_2.fit(train_features, train_target)

        cbc_3 = CatBoostClassifier(**params)
        train_target = df['theme_3']
        cbc_3.fit(train_features, train_target)

        th1_predictions = cbc_1.predict(test_df)
        th2_predictions = cbc_2.predict(test_df)
        th3_predictions = cbc_3.predict(test_df)
        
> Так как задача у нас была в интерактивном режиме, то вывод мы просто печатаем. В качестве вывода у нас просто порядковые номера записей из тестовой выборки которые соответсвую трем классам. По порядку. 
              
        th_1_index = np.where(th1_predictions == 1)
        th_2_index = np.where(th2_predictions == 1)
        th_3_index = np.where(th3_predictions == 1)

        print (len(th_1_index[0]) , len(th_2_index[0]) ,  len(th_3_index[0]))
        for x in th_1_index[0]:
            print (str(x))
        for x in th_2_index[0]:
            print (str(x))
        for x in th_3_index[0]:
            print (str(x))

> Финальный скор этого решения умноженный на 10 000 - 5099.61 и 4-ое место. 

________________________________________________________________________________________________

> P.S. Помимо призовых и различных промокодов, топ-20 финалистов награждяются еще и Fast Track'ом в Яндекс. Это возможность существенно сократить путь трудоустройства до всего двух техничестких собеседований. Ниже будет совет тем, кто, также как и я, планирует участие в чемпионате именно для того, чтобы устроиться на работу в Яндекс.

> Два собеседования которые мне предстояли это "алгоритмическая секция" и "секция по машинному обучению". Подробную информацию вы можете найти в сети, поэтому не буду ничего про них рассказывать. Но хочу рассказать о своей ошибке, чтобы предостеречь других соискателей. 

> Время собеседований вы вольны выбирать сами. Fast Track действует один год. Как то так случилось, что я этого не знал. Знал только что предложение ограничено по времени. Это подтолкнуло меня к тому, что не нужно особо затягивать со сроками прохождения собеседований. Ну и в результате не очень хорошо подготовился, волновался и завалил первое же - "алгоритмы". Вроде бы не страшно - сам Яндекс везде пишет что можно пробовать еще, после того как подготовишься получше. Но! Дело в том, что результаты прохождения конкретного собеседования тоже действительны год! Таким образом, перепройти "Алгоритмы" у меня возможность появится только тогда, когда истечет срок текущего Fast Track'а! 

> В общем сплошное расстройство. Возможно для кого то этот совет окажется самым ценным во всем этом тексте. Для меня бы точно так и было! 

>> Не спешите проходить собеседования! У вас полно времени на подготовку, а в случае одного провала весь бонус от участия в чемпионате превращается в тыкву. 
