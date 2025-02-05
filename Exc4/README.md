##  Решение

1. Сервис ***core-app*** принимает заявку на оа ОСАГО от клиента и отправляет через брокер задание в ***osago-aggregator*** на отправку заявки клиента в страховые компании и дальнейший опрос. Выбран асинхронный вариант взаимодействия между ***osago-aggregator*** и ***core-app*** так как операция довольно длительная и тяжелая и нет необходимости с синхронном варианте учитывая что у нас ожидается рост числа старховых компаний. Также асинхронный вариант больше в случае когда экземпляров сервисов будет больше одного. В таком варианте задание примет только один сервис.
2. Сервис ***osago-aggregator*** принимает задание от ***core-app*** и отправляет заявку в страховые компании. После этого начинается опрос страховых компаний по поводу решения по заявке. После того как будет получено предложение от страховой компании прдложение будет отправлено сервису ***core-app***. Также можно рассмотреть вариант при котором предложение по заявке от старховой компании сохранаяется в базе ***osago-aggregator*** а в ***core-app*** отправляется событие и в теле события указываем ссылку на предложение. Но если размер заявки не слишком большой то лучше сразу само предложение отправлять через брокер как документ. Особенности выполнения задания:
    1. Сервис принимает задание и начинает его выполнение. Для надежности данные по заданию нужно сохранить в базе данных чтобы само задание и его статус не потерялись.
    2. Отправляет в каждую страховую компанию заявку. Если запрос неудачен то следует делать повторы(Retry, например максимум 10 раз).
    3. После успешной отправки заявки начинаем опрос страховой компании чтобы получить статус заявки. Общий таймаут 60 секунд. Если в течении данного интервала предложение не сфрмировано то исключаем данную старховую компанию.
    4. После того все старховые компании опрошены(удачно или нет) и результаты отправлены ***core-app***, то помечаем в базе задание как выполненное.
3. Сервис ***core-app*** отобразит предложения клиенту

## Детали
1. Сервису ***osago-aggregator*** нужна своя база данных чтобы хранить данные по заданию
2. Так как серисы ***core-app*** и ***osago-aggregator*** взаимодействуют через брокер то специального REST API для ***core-app*** предоставлять не надо.
3. ***core-app*** будет предоставлять REST API для веб приложения:
    1. Для создания заявки
    2. Для отрображения предложений по заявке(кам)
    3. Оформление заявки
    4. Отбразить статус заявки
4. Для API для веб кприложения(для клиентов) следует внедрить rate limit исходя из прогноза 2.500 человек в пике
5. Retry и timeout следует внедрить в операциях взаимодействия с API старховых компаний. Причем отдельный таймаут на любой REST запрос. И общий таймаут на операцию получения предложения.