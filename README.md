# testMD
Диагностика связи с АРМом
___

1. Запустите Среду разработки «Studio» и подключитесь к серверу. 
1. В окне серверных объектов «Server Explorer», в корневом каталоге «Tags(Теги)» раскройте папку с названием проекта и перейдите в папку «Workstations». 
1. Найдите тег «LastActiveTime»
1. Нажмите ПКМ на тег «LastActiveTime» и в контекстном меню выберите опцию контектсного меню  «Permissions...». 
1. В открывшемся окне «Permissions for LastActiveTime» предоставьте всем пользователям доступ для записи в колонке «Write» и нажмите кнопку «Close»:

   <img src="images/permissionsLastActiveTime.png" alt=""/>

Среда исполнения «Studio» от имени текущего пользователя в тег «LastActiveTime» пишет текущее время в тиксах, с периодом указанным в теге «HeartbeatPeriod». Если разница между текущим значением времени и предыдущим больше значения тега «HeartbeatPeriod» это означает, что среда исполнения не запущена.

Минимальное значение тега «HeartbeatPeriod» - 1 секунда, максимальное - 600 секунд.

В корневаой папке «Jobs(Программы)» (в окне серверных объектов «Server Explorer») создайте папку с именем проекта, в которой создайте программу для генерации аларма. Пример программы для генерации аларма об отсутствии связи с АРМом представлен в листинге 1:

```lua
-- имя папки с настройками среды исполнения
local wkName ='Main'
local function getJobFolder()
local folder =' '
local job =' '
for str in string.gmatch(Context:GetPath(), "[^/]+") do
    folder = job
    job = str
end
return job, folder
end
local tps = 10 ^ 7
local function getTimezoneOffset(ts)
local utcdate = os.date("!*t", ts)
local localdate = os.date("*t", ts)
localdate.isdst = false -- this is trick
return os.difftime(os.time(localdate), os.time(utcdate))
end
-- настройки
local job, prjName = getJobFolder()
-- инициализация
local root = Context:GetTagFolder( '/ ')
if root == nil then return print('Нет␣корневой␣папки␣с␣тегами') end
prjFolder = root:GetFolder(prjName)
if prjFolder == nil then return root:ReportEvent(prjName, 800, 'Нет␣папки␣с␣тегами для␣проекта␣'..prjName) end
local wksFolder = prjFolder:GetFolder('Workstations')
if wksFolder == nil then return root:ReportEvent(prjName, 800, 'Нет␣папки␣с␣тегами Workstations␣для␣проекта␣'..prjName) end
local wkFolder = wksFolder:GetFolder(wkName)
if wkFolder == nil then return root:ReportEvent(prjName, 800, 'Нет␣папки␣с␣тегами конфигурации␣рантайма␣'..wkName..'␣в␣папке␣Workstations␣для␣проекта␣'..prjName) end
local wksActiveTag = wkFolder:GetTag('LastActiveTime ')
local alarmWksActive = root:GetAlarm(prjName)
if wksActiveTag == nil then
if not alarmWksActive.Enabled then alarmWksActive:Enable(prjName, 500, 'Нет␣тега LastActiveTime␣для␣конфигурации␣рантайма␣'..wkName..'␣для␣проекта␣'..prjName) end
-- для конфигурации 'main'
return
else
if alarmWksActive.Enabled then alarmWksActive:Disable(100, 'Тег␣LastActiveTime добавлен␣для␣конфигурации␣рантайма␣'..wkName..'␣для␣проекта␣'..prjName) end
end
if wksActiveTag.Value == nil then return end
local wksHeartBeat = wkFolder:GetTag('HeartbeatPeriod')
local interval
local default = 10 * tps
local alarmInterval = wkFolder:GetAlarm('HeartbeatPeriod')
if wksHeartBeat == nil or wksHeartBeat.Value == nil then
if not alarmInterval.Enabled then alarmInterval:Enable(prjName, 500, 'Отсутствует␣тег HeartbeatPeriod,␣либо␣не␣задан␣интервал') end
interval = default
elseif wksHeartBeat.Value < 10 or wksHeartBeat.Value > 600 then
if not alarmInterval.Enabled then alarmInterval:Enable(prjName, 500, 'Значение HeartbeatPeriod␣больше␣10␣минут␣или␣меньше␣10␣секунд') end
interval = default
else
if alarmInterval.Enabled then alarmInterval:Disable(100, 'Значение␣Heartbeat-Period ␣в␣пределах␣интервала') end
interval = wksHeartBeat.Value * tps
end
local wksTime = wksActiveTag.Value + (getTimezoneOffset(os.time())*tps)
local wksAlarm = wksFolder:GetAlarm(wksFolder.DisplayName)
if interval < (os.time() - wksTime) and not wksAlarm.Enabled then wksAlarm:Enable(wksFolder.DisplayName, 700, 'Нет␣связи␣с␣АРМ␣Оператора')
    else
        if interval >= (os.time() - wksTime) and wksAlarm.Enabled then wksAlarm:Disable(100, 'Связь␣с␣АРМ␣Оператора␣восстановлена')
        end
end
```

Созданную программу необходимо запускать по расписанию каждые 10 секунд.

> [!div class="nextstepaction"]
> [В начало раздела](asdue_ck_power.md)
___
