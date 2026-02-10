-- HELL CLICKER: HARDCORE 45 STAGES (No Reset Mode)
local HttpService = game:GetService("HttpService")
local Player = game.Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
local FileName = "HellHardcoreNoReset.txt"

-- Сокращение чисел
local function Format(n)
    local signs = {"", "K", "M", "B", "T", "Qd", "Qi", "Sx", "Sp", "Oc", "No", "Dc"}
    local i = 1
    while n >= 1000 and i < #signs do
        n = n / 1000
        i = i + 1
    end
    return (math.floor(n * 100) / 100) .. signs[i]
end

-- Загрузка данных
local function LoadData()
    local default = {Money = 0, Best = 0, Multi = 1, Auto = 0, Unlocked = 0}
    if isfile and isfile(FileName) then
        local success, result = pcall(function() return HttpService:JSONDecode(readfile(FileName)) end)
        if success and result then return result end
    end
    return default
end

local data = LoadData()
_G.HC_Money = data.Money
_G.HC_Best = data.Best
_G.HC_Multi = data.Multi
_G.HC_Auto = data.Auto
_G.HC_Unlocked = data.Unlocked or 0

-- Сохранение данных
local function SaveData()
    local payload = {
        Money = _G.HC_Money,
        Best = _G.HC_Best,
        Multi = _G.HC_Multi,
        Auto = _G.HC_Auto,
        Unlocked = _G.HC_Unlocked
    }
    writefile(FileName, HttpService:JSONEncode(payload))
end

-- Создание UI
if PlayerGui:FindFirstChild("HellNoReset") then PlayerGui.HellNoReset:Destroy() end
local ScreenGui = Instance.new("ScreenGui", PlayerGui); ScreenGui.Name = "HellNoReset"

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 350, 0, 500); Main.Position = UDim2.new(0.5, -175, 0.5, -250)
Main.BackgroundColor3 = Color3.fromRGB(15, 0, 0); Main.Active = true; Main.Draggable = true
Instance.new("UICorner", Main)

-- Кнопка X
local Close = Instance.new("TextButton", Main)
Close.Size = UDim2.new(0, 35, 0, 35); Close.Position = UDim2.new(1, -40, 0, 5)
Close.Text = "X"; Close.BackgroundColor3 = Color3.fromRGB(180, 0, 0); Close.TextColor3 = Color3.new(1,1,1)
Close.Font = "GothamBold"; Close.MouseButton1Click:Connect(function() SaveData(); ScreenGui:Destroy() end)
Instance.new("UICorner", Close)

-- Рекорд
local BestL = Instance.new("TextLabel", Main)
BestL.Size = UDim2.new(0, 200, 0, 30); BestL.Position = UDim2.new(0, 10, 0, 5)
BestL.Text = "RECORD: $" .. Format(_G.HC_Best); BestL.TextColor3 = Color3.fromRGB(255, 200, 0)
BestL.BackgroundTransparency = 1; BestL.Font = "GothamBold"; BestL.TextXAlignment = "Left"

-- Баланс
local ML = Instance.new("TextLabel", Main)
ML.Size = UDim2.new(1, 0, 0, 50); ML.Position = UDim2.new(0, 0, 0, 40)
ML.Text = "$" .. Format(_G.HC_Money); ML.TextColor3 = Color3.new(1, 1, 1); ML.TextSize = 35; ML.Font = "GothamBold"; ML.BackgroundTransparency = 1

-- Кнопка клика
local Click = Instance.new("TextButton", Main)
Click.Size = UDim2.new(0, 100, 0, 100); Click.Position = UDim2.new(0.5, -50, 0, 100)
Click.Text = "TAP"; Click.BackgroundColor3 = Color3.fromRGB(200, 0, 0); Click.TextColor3 = Color3.new(1,1,1); Click.Font = "GothamBold"
Instance.new("UICorner", Click).CornerRadius = UDim.new(1, 0)

-- Магазин
local Shop = Instance.new("ScrollingFrame", Main)
Shop.Size = UDim2.new(1, -20, 0, 280); Shop.Position = UDim2.new(0, 10, 1, -290)
Shop.BackgroundTransparency = 1; Shop.CanvasSize = UDim2.new(0, 0, 25, 0); Shop.ScrollBarThickness = 4
Instance.new("UIListLayout", Shop).Padding = UDim.new(0, 5)

-- Генерация 45 ступеней
for i = 1, 45 do
    local isAuto = (i % 2 == 0)
    local price = 100 * (10 ^ (i * 1.15)) 
    local power = 10 * (8 ^ (i * 0.75))
    
    if i > _G.HC_Unlocked then
        local btn = Instance.new("TextButton", Shop)
        btn.Size = UDim2.new(1, -10, 0, 50); btn.BackgroundColor3 = Color3.fromRGB(35, 5, 5)
        btn.TextColor3 = Color3.new(1,1,1); btn.Font = "SourceSansBold"; btn.TextSize = 13
        btn.Text = "STAGE "..i..": "..(isAuto and "Auto" or "Click").."\nCost: $"..Format(price).." | +"..Format(power)
        Instance.new("UICorner", btn)
        
        btn.MouseButton1Click:Connect(function()
            if _G.HC_Money >= price then
                _G.HC_Money = _G.HC_Money - price
                _G.HC_Unlocked = i
                if isAuto then _G.HC_Auto = _G.HC_Auto + power else _G.HC_Multi = _G.HC_Multi + power end
                btn:Destroy()
                SaveData()
            end
        end)
    end
end

-- Основной цикл
task.spawn(function()
    local lastSave = tick()
    while ScreenGui.Parent do
        _G.HC_Money = _G.HC_Money + (_G.HC_Auto / 10)
        ML.Text = "$" .. Format(_G.HC_Money)
        
        if _G.HC_Money > _G.HC_Best then 
            _G.HC_Best = _G.HC_Money 
            BestL.Text = "RECORD: $" .. Format(_G.HC_Best)
        end
        
        if tick() - lastSave > 10 then
            SaveData()
            lastSave = tick()
        end
        task.wait(0.1)
    end
end)

Click.MouseButton1Click:Connect(function()
    _G.HC_Money = _G.HC_Money + _G.HC_Multi
    Click:TweenSize(UDim2.new(0, 90, 0, 90), "Out", "Quad", 0.05, true)
    task.wait(0.05); Click:TweenSize(UDim2.new(0, 100, 0, 100), "Out", "Quad", 0.05, true)
end)
