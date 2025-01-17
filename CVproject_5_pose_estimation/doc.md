#### Запуск моделей OpenPose и ResNet осуществлен в целях тестирования на пользовательских данных. Входные данные – видеофайлы формата .mp4, .avi.

#### Обучение и валидация модели упущены, так как выполнены разработчиками на датасете Leeds Sports Poses. Поэтому исходные данные не претерпевают трансформаций, необходимых для обучения.

## ___Этапы реализации OpenPose___
1.   Загрузка модели с github
2.   Запуск модели на видео с коучем и на видео со студентом со следующими гиперпарметрами:
     * Масштабирование координат опорных точек до размера выходного видео (keypoint_scale 1)
     * Размер выходного видео (net_resolution '656x368')
     * Также указаны путь сохранения видео с визуализированным скелетом и путь сохранения файлов с координатами опорных точек.
3.   Форматирование полученного результата в .mp4.
4.   Для адекватного сравнения видео и расчета метрик:
     * Отобрано по одному фрейму из каждой секунды видео, так как каждое видео имеет разную частоту.
     * Из отобранных фрэймов сохранены координаты опорных точек и вероятность их правильного определения.
     * Для каждой секунды удалены координаты тех опорных точек, которые не были определены или у коуча или у студента. Это сделано для избежания сопоставления координаты с нулевым значением.
     * Аффиные преобразования на данном этапе не выполнялись, так как они уже включены в код OpenPose.
     * Удаление фрэймов, где обнаружено более одного человека, не потребовалось. На таких фреймах детектированы только лица, поэтому при отборе ключевых точек выбирался набор с наибольшим количеством определенных координат.
5.   Для каждой секунды видео рассчитана матрица косинусного сходства между всеми отобранными опорными точками коуча и всеми отобранными опорными точками студента. Для оценки позы рассчитана средняя величина диагональных элементов полученной матрицы.
6.   Для каждой секунды видео рассчитано взвешенное совпадение, которое в абсолютном отношении не имеет смысла, а лишь показывает, какая поза оказалось максимально приближенной к эталонной.
7.   Видео студента, полученное в итоге реализации модели, перезаписано с нанесением на фреймы, участвующие в сопоставлении, метрик качества поз.
8.   Из видео студента сохранен каждый фрейм из секундного интервала в формате изображения.
9.   Каждое шестое сохраненное изображение сопоставлено с параллельным фреймом из видео коуча. Оба кадра размещены бок о бок и сохранены в формате изображения.
10.   Посчитаны обобщенные метрики по всему видео (модальное косинусное сходство и модальное взвешенное совпадение), а также статистика по количеству опорных точек, удаленных в целях качественного расчета метрик. 

#### ___Итоги реализации OpenPose___  
Модальное косинусное сходство ___98.32%___  
Модальное взвешенное совпадение ___50___  
Обработано ___100%___ сохраненных фреймов  
Удален ___21%___ опорных точек. Это объясняется включением в модель опорных точек пяток и низким уровнем их детекции по причине близко расположенного объекта к нижней границе кадра.

## ___Этапы реализации ResNet-50 FPN___
1.   Кадрирование исходных данных. Захват одного фрейма из каждой секунды видео
2.   Загрузка предобученной модели с весами ResNet50_FPN.
3.   Трансформирование каждого изображения и передача модели для предсказания опорных точек.
4.   Для адекватного расчета метрик для каждой пары фотографий коуча/студента:
     * Удалены координаты опорных точек, предсказанный с низким значением вероятности
     * Удалены координаты тех опорных точек, которые не были определены или у коуча или у студента. Это сделано для избежания сопоставления координаты с нулевым значением.
     * Сохранены индексы фотографий, где или на фото коуча или на фото студента не детектировано ни одного человека с высокой степенью вероятности.
     * Сохранены индексы фотографий, где или на фото коуча или на фото студента детектировано более одного человека с большой степенью вероятности. Это сделано для универсальности проведения расчётов, так как задача не предполагает детекцию нескольких человек на изображении. Выбор наиболее адекватного и полного набора опорных точек не реализован, так как таких изображений ни так много.
     * По сохраненным индексам сопоставление изображений не проводилось.
     * Для дальнейших расчётов сохранены опорные точки коуча, опорные точки студента, координаты ограничивающей рамки коуча, константы ключевых точек для каждого изображения с учётом удаления координат отдельных опорных точек, а также номера фото, участвующих в оценке и исключенных из оценки.
5.   Выполнены аффиные преобразования.
6.   Для каждой секунды видео рассчитана матрица косинусного сходства между всеми отобранными опорными точками коуча и всеми отобранными опорными точками студента. Для оценки позы рассчитана средняя величина диагональных элементов полученной матрицы.
7.   Для каждого изображения рассчитано сходство ключевых точек как альтернатива взвешенному совпадению. Отказ от взвешенного совпадения принят в силу того, что оценка уверенности определения ключевых точек в моделе ResNet не ограничивается интервалом от 0 до 1.
9.   Изображения коуча и студента размещены бок о бок и сохранены в отдельном файле. Предварительно на каждом изображении визуализирован детектированный скелет, на фото студента нанесены рассчитанные метрики. На изображения, исключенные из оценки, нанесено сообщение об ошибке. 
10.   Из всех изображений студента с визуализированным скелетом и нанесенными метриками записан видео-файл с частотой 2 кадра в секунду.
11.   Посчитаны обобщенные метрики по всему видео (модальное косинусное сходство и модальное сходство ключевых точек), а также статистика по количеству обработанных фреймов и количеству опорных точек, удаленных в целях качественного расчета метрик. 

#### ___Итоги реализации ResNet-50 FPN___  
Модальное косинусное сходство ___98.86%___  
Модальное сходство ключевых точек ___100.00%___  
Обработано ___93%___ сохраненных фреймов  
Удалено ___15%___ опорных точек из обработанных фреймов.
