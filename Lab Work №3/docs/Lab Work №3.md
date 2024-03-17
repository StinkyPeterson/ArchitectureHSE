# Лабораторная работа 3

## 1. Диаграмма компонентов 
Сервис идентификации и работы с иногородними пациентами

![image](component.jpg)

## 2. Диаграмма вариантов использования работы с иногородними пациентами

![image](diagram.jpg)

Пояснения диаграммы использования:

1. Поиск пациента:
    * Пользователь вводит критерии поиска пациентов (ФИО, дата рождения, полис и т.д.)
    * Controller делает запрос Repository согласно заданным фильтрам, Repository обращается к БД для поиска пациентов
    * Ситема отображает пользователю отфильтрованный список пациентов
2. Добавление пациента:
    * Пользователь заполняет необходимые данные для добавления нового пациента
    * Service проводит валидация СМО (не должен начинаться с кода региона, так как пациент - иногородний) и номер полиса (16 символов)
    * В случае если данные не прошли валидацию, возвращается соотвествующее сообщение
    * Service отправляет запрос Repository на получение пациента по данным (ФИО, дата рождения)
    * Если такой пациент уже присутствует в БД, система уведомляет пользователя, что такой пациент уже присутствует в БД и не создает новую запись
    * В случае валидности данных и если пациента нет в БД, система добавляет нового пациента в БД, возвращается сообщение о добавлении пациента
3. Удаление пациента:
    * Пользователь отправляет запрос на удаление пациента 
    * Система удаляет выбранного пациента из БД
    * Система возвращает сообщение о удалении пациента
4. Обновление пациента:
    * Пользователь отправляет запрос на редактирование пациента
    * Service проводит валидацию СМО и номера полиса
    * Система обновляет данные пациента в БДй
    * Система уведомляет пользователя о статусе редактирования пациента

## 3. Модель базы данных

![image](db.jpg)

* База данных содержит всего 1 модель иногороднего пациента.

## 4. Применение основных принципов разработки

    
#### Контроллер
    public class IngorodController : ODataController
    {
        private readonly IIngorodPatientService _ingorodPatientService;
        public IngorodController(IIngorodPatientService ingorodPatientService)
        {
            _ingorodPatientService = ingorodPatientService;
        }
        [HttpGet]
        [EnableQuery]
        public async Task<List<IngorodPatient>> Get()
        {
            return await _ingorodPatientService.GetAllPatients();
        }

        [HttpPost]
        public async Task<IActionResult> Post([FromBody] IngorodPatient patient)
        {
            ResponseDto dto = new ResponseDto();
            OperationResult<string> result = new OperationResult<string>();
            result = await _ingorodPatientService.CreateOrUpdatePatient(patient);
            ...
            
            ...
            // обработка результата
            return Ok(dto)
        }

        [HttpDelete("api/[controller]({id})")]
        public async Task<IActionResult> Delete(string id)
        {
            ResponseDto dto = new ResponseDto();
            bool result = await _ingorodPatientService.DeletePatient(id);
            ...

            ...
            // обработка результата
            return Ok(dto)

        }

        [HttpPatch("api/[controller]({id})")]
        public async Task<IActionResult> Patch(string id, [FromBody] IngorodPatient patient)
        {
            patient.Id = id;
            ResponseDto dto = new ResponseDto();
            OperationResult<string> result = new OperationResult<string>();
            result = await _ingorodPatientService.CreateOrUpdatePatient(patient);
            ...

            ...
            // обработка результата
            return Ok(dto)
        }
    }

#### Интерфейс сервиса и его реализация

    public interface IIngorodPatientService
    {
        public Task<List<IngorodPatient>> GetAllPatients();
        public Task<OperationResult<string>> CreatePatient(IngorodPatient ingorodPatient);
        public Task<OperationResult<string>> UpdatePatient(IngorodPatient ingorodPatient);
        public Task<bool> DeletePatient(string id);
    }

    public class IngorodPatientService : IIngorodPatientService
    {
        private IIngorodRepository _repository;
        private ILogger<IngorodPatientService> _logger;
        private IIngorodPatientValidator _ingorodPatientValidator;
        public IngorodPatientService(IIngorodRepository repository, ILogger<IngorodPatientService> logger, IIngorodPatientValidator ingorodPatientValidator)
        {
            _repository = repository;
            _logger = logger;
            _ingorodPatientValidator = ingorodPatientValidator;
        }


        public async Task<bool> DeletePatient(string id)
        {
            return await _repository.DeleteById(id);
        }

        public async Task<List<IngorodPatient>> GetAllPatients()
        {
            return await _repository.GetAll();
        }
        
        public async Task<OperationResult<string>> CreateOrUpdatePatient(IngorodPatient ingorodPatient)
        {
            OperationResult<string> result = new OperationResult<string>();
            try
            {
                //валидация данных
                bool isValidate = _ingorodPatientValidator.ValidateIngorodPatient(ingorodPatient);
                if (!isValidate)
                {
                    result.AddFail("Пациент не прошел валидацию", OperationInfo.Warning);
                    return result;
                }

                ...

                ...
                //логика добавления или обновления пациента

                return result;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex.Message, ex);
                result.AddFail("Неизвестная ошибка", OperationInfo.Error);
                return result;
            }
        }

    }

#### Интерфейс репозитория и его реализация

    public interface IIngorodRepository
    {
        public Task<List<IngorodPatient>> GetAll();
        public Task<IngorodPatient> GetById(string id);
        public Task<bool> InsertOne(IngorodPatient patient);
        public Task<bool> UpdateOne(IngorodPatient patient);
        public Task<bool> DeleteById(string id);
    }

    public class IngorodRepository : IIngorodRepository
    {
        private readonly DbContextIngorodPatient _dbContext;
        private readonly ILogger<IngorodRepository> _logger;

        public IngorodRepository(DbContextIngorodPatient dbContext, ILogger<IngorodRepository> logger)
        {
            _dbContext = dbContext;
            _logger = logger;
        }

        public async Task<List<IngorodPatient>> GetAll()
        {
            return await _dbContext.IngorodPatients.ToListAsync();
        }

        public async Task<IngorodPatient> GetById(string id)
        {
            return await _dbContext.IngorodPatients.FirstOrDefaultAsync(c => c.Id == id);
        }

        public async Task<bool> InsertOne(IngorodPatient ingorodPatient)
        {
            try
            {
                await _dbContext.AddAsync(ingorodPatient);
                await _dbContext.SaveChangesAsync();
                return true;
            }
            catch (Exception e)
            {
                _logger.LogError(e.Message, e);
                return false;
            }
        }

        public async Task<bool> UpdateOne(IngorodPatient ingorodPatient)
        {
            try
            {
                // логика обновления данных пациента
                ... 

                ...
                return true;
            }
            catch (Exception e)
            {
                return false;
            }
        }
        public async Task<bool> DeleteById(string id)
        {
            try
            {
                await _dbContext.IngorodPatients.Where(c => c.Id == id).ExecuteDeleteAsync();
                return true;
            }
            catch(Exception e)
            {
                return false;
            }   
        }
    }


1. KISS (Keep It Simple, Stupid):
Принцип KISS гласит, что программное обеспечение должно быть максимально простым, понятным и не содержать избыточных элементов. Контроллер `IngorodController` представляет простые методы для работы с иногородними пациентами, скрывая сложность реализации в `IngorodPatientService`.

2. DRY (Don't Repeat Yourself)
Принцип DRY говорит о том, что необходимо избегать дублирования кода. Логика валидации данных пациента и обработки результата вынесены в отдельный метод, что позволяет избежать повторения кода

3. SOLID (Принципы SOLID)
Принципы SOLID - это набор принципов проектирования ПО, которые делают код более понятным, гибким и легко расширяемым.
    * Принцип Единственной Ответственности (Single Responsibility Principle):

        * Контроллер `IngorodController` отвечает за обработку HTTP запросов и вызовы сервиса для выполнения операций с пациентами. Сервис `IngorodPatientService` отвечает за бизнес-логику операций с пациентами и валидацию данных.
    * Принцип Открытости/Закрытости (Open/Closed Principle):
        * Код не является слишком закрытым для расширения. Например, при добавлении новых операций над пациентами или новых проверок данных, необходимо будет изменить соответствующие методы в сервисе `IngorodPatientService`, но это вполне ожидаемо для изменения бизнес-логики.
    * Принцип Подстановки Барбары Лисков (Liskov Substitution Principle):
        * Принцип LSP говорит о том, что объекты одного класса могут быть заменены объектами другого подкласса без изменения свойств программы. В данном коде не используется
    * Принцип Разделения Интерфейса (Interface Segregation Principle):
        * В коде нет явного использования интерфейсов, которые могли бы разделить большие интерфейсы на более мелкие и специализированные. Но этот принцип может быть не так важен в случае с контроллерами и сервисами.
    * Принцип инверсии зависимостей (Dependency Inversion Principle):
        * Классы `IngorodController` и `IngorodPatientService` используют интерфейсы для взаимодействия с зависимостями (`DbContextIngorodPatient`, `IIngorodRepository`, `ILogger`, `IIngorodPatientValidator`). Это соответствует принципу DIP.

4. YAGNI (You Aren't Gonna Need It):
Принцип YAGNI утверждает, что не нужно добавлять в код функциональность, которая не требуется на текущий момент:
    * Контроллер IngorodController:
        * Контроллер содержит только методы для базовых операций CRUD. Это соответствует принципу YAGNI, поскольку реализует только то, что необходимо на данный момент.
    * Сервис IngorodPatientService:
        * Сервис содержит методы для выполнения операций с пациентами и валидации данных. Это тоже соответствует принципу YAGNI, так как содержит только нужные функции.