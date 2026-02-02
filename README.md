local Players = game:GetService("Players")
local player = Players.LocalPlayer
local runService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

-- Variáveis Únicas
local autoFarmRadioactive = false
local autoFarmGold = false
local antiAfkActive = false
local flyActive = false
local flySpeed = 50 
local itemName = "GoldBar"
local itemName = "Radioactive Coin"

-- [SISTEMA DE VOO]
local function startFly()
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local hum = char:WaitForChild("Humanoid")
    
    for _, v in pairs(hrp:GetChildren()) do
        if v.Name == "IYFlyVelocity" or v.Name == "IYFlyGyro" then v:Destroy() end
    end

    local velocity = Instance.new("BodyVelocity")
    velocity.Name = "IYFlyVelocity"
    velocity.Parent = hrp
    velocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    velocity.Velocity = Vector3.new(0, 0, 0)

    local gyro = Instance.new("BodyGyro")
    gyro.Name = "IYFlyGyro"
    gyro.Parent = hrp
    gyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    gyro.CFrame = hrp.CFrame
    hum.PlatformStand = true 

    task.spawn(function()
        while flyActive and char.Parent do
            local camera = workspace.CurrentCamera
            if hum.MoveDirection.Magnitude > 0 then
                velocity.Velocity = camera.CFrame.LookVector * flySpeed
            else
                velocity.Velocity = Vector3.new(0, 0, 0)
            end
            gyro.CFrame = camera.CFrame
            runService.Heartbeat:Wait()
        end
        if velocity then velocity:Destroy() end
        if gyro then gyro:Destroy() end
        if hum then hum.PlatformStand = false end
    end)
end

-- [SISTEMA DE TELEPORTE]
local function teleportarMoedas()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    
    local fila = {}
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj.Name == itemName and obj:IsA("BasePart") then
            if obj:FindFirstAncestor("EventParts") or obj.Parent.Name == itemName then
                table.insert(fila, obj)
            end
        end
    end

    for _, moeda in ipairs(fila) do
        if not autoFarmActive then break end
        if moeda and moeda.Parent then
            moeda.CanCollide = false
            moeda.CFrame = hrp.CFrame
            task.wait(0.01)
        end
    end
end

-- [INTERFACE VERMELHO HUB]
local function criarVermelhoHub()
    if player.PlayerGui:FindFirstChild("VermelhoHubGui") then player.PlayerGui.VermelhoHubGui:Destroy() end

    local sg = Instance.new("ScreenGui", player.PlayerGui)
    sg.Name = "VermelhoHubGui"
    sg.ResetOnSpawn = false

    local main = Instance.new("Frame", sg)
    main.Size = UDim2.new(0, 420, 0, 280)
    main.Position = UDim2.new(0.5, -210, 0.5, -140)
    main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    main.Active = true
    main.Draggable = true 
    Instance.new("UICorner", main)

    -- Botão V
    local minBtn = Instance.new("TextButton", sg)
    minBtn.Size = UDim2.new(0, 50, 0, 50)
    minBtn.Position = UDim2.new(0, 10, 0.5, -25)
    minBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    minBtn.Text = "V"
    minBtn.TextColor3 = Color3.fromRGB(255, 0, 0)
    minBtn.Font = "GothamBold"
    minBtn.TextSize = 25
    minBtn.Active = true
    minBtn.Draggable = true 
    Instance.new("UICorner", minBtn).CornerRadius = UDim.new(1, 0)
    minBtn.MouseButton1Click:Connect(function() main.Visible = not main.Visible end)

    local sidebar = Instance.new("Frame", main)
    sidebar.Size = UDim2.new(0, 100, 1, -35)
    sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    Instance.new("UICorner", sidebar)
    Instance.new("UIListLayout", sidebar).HorizontalAlignment = "Center"

    local content = Instance.new("ScrollingFrame", main)
    content.Position = UDim2.new(0, 110, 0, 10)
    content.Size = UDim2.new(1, -120, 1, -50)
    content.BackgroundTransparency = 1
    content.ScrollBarThickness = 2
    Instance.new("UIListLayout", content).Padding = UDim.new(0, 10)

    local function clearContent()
        for _, v in pairs(content:GetChildren()) do if not v:IsA("UIListLayout") then v:Destroy() end end
    end

    local function createToggle(text, callback, state)
        local btn = Instance.new("TextButton", content)
        btn.Size = UDim2.new(0.95, 0, 0, 40)
        btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35)
        btn.Text = text .. (state and ": ON" or ": OFF")
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Font = "Gotham"
        Instance.new("UICorner", btn)
        btn.MouseButton1Click:Connect(function()
            state = not state
            btn.Text = text .. (state and ": ON" or ": OFF")
            btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35)
            callback(state)
        end)
    end

    local function addTab(name, func)
        local b = Instance.new("TextButton", sidebar)
        b.Size = UDim2.new(0.9, 0, 0, 35)
        b.Text = name; b.BackgroundColor3 = Color3.fromRGB(30, 30, 30); b.TextColor3 = Color3.new(1, 1, 1); b.Font = "GothamBold"
        Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function() clearContent(); func() end)
    end

    -- Criação das Abas
    local function tabEvento()
        clearContent()
        createToggle("Farm GoldBar", function(v) autoFarmGold = v end, autoFarmGold)
    end

local function coletarRadioactive()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local hrp = char.HumanoidRootPart
    
    local fila = {}
    
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj.Name == itemName and obj:IsA("BasePart") then
            -- Verificação de segurança (EventParts) vinda do segundo script
            if obj:FindFirstAncestor("EventParts") or obj.Parent.Name == itemName then
                table.insert(fila, obj)
            end
        end
    end

    -- Parte vinda do script de execução (Teleporte)
    for _, moeda in ipairs(fila) do
        if not farmRadioactiveActive then break end
        if moeda and moeda.Parent then
            moeda.CanCollide = false -- Garante que não bata em nada
            moeda.CFrame = hrp.CFrame
            task.wait(0.01) -- Velocidade de processamento
        end
    end
end

task.spawn(function()
    while true do
        if autoFarmRadioactive then
            pcall(function()
                coletarRadioactive()
            end)
        end
        task.wait(0.5) -- Intervalo entre as buscas no mapa
    end
end)

-- createToggle("Farm Radioactive", function(v) 
--    autoFarmRadioactive = v 
-- end, autoFarmRadioactive)

    
    local function tabMisc()
        clearContent()
        createToggle("Fly (Voo)", function(v) flyActive = v; if v then startFly() end end, flyActive)
        createToggle("Anti-AFK", function(v) antiAfkActive = v end, antiAfkActive)
    end

    local function tabFPS()
        clearContent()
        local b = Instance.new("TextButton", content)
        b.Size = UDim2.new(0.95, 0, 0, 40); b.Text = "Ativar Anti-Lag"; b.BackgroundColor3 = Color3.fromRGB(30, 30, 30); b.TextColor3 = Color3.new(1, 1, 1)
        Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function()
            for _, v in pairs(game:GetDescendants()) do
                if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic elseif v:IsA("Decal") then v:Destroy() end
            end
        end)
    end

    addTab("Evento", tabEvento)
    addTab("Misc", tabMisc)
    addTab("FPS", tabFPS)

    tabEvento() -- Abre na aba de evento por padrão
end

-- [INTERFACE REDZ STYLE]
local function criarRedzMenu()
    if player.PlayerGui:FindFirstChild("RedzStyleMenu") then player.PlayerGui.RedzStyleMenu:Destroy() end

    local sg = Instance.new("ScreenGui", player.PlayerGui)
    sg.Name = "RedzStyleMenu"
    sg.ResetOnSpawn = false

    local main = Instance.new("Frame", sg)
    main.Size = UDim2.new(0, 220, 0, 150)
    main.Position = UDim2.new(0.5, -110, 0.05, 0) -- Posição no topo
    main.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
    main.Active = true
    main.Draggable = true
    Instance.new("UICorner", main)

    local btnAuto = Instance.new("TextButton", main)
    btnAuto.Size = UDim2.new(0.9, 0, 0, 50)
    btnAuto.Position = UDim2.new(0.05, 0, 0.4, 0)
    btnAuto.Text = "AUTO-FARM: OFF"
    btnAuto.TextColor3 = Color3.new(1, 1, 1)
    btnAuto.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btnAuto.Font = "GothamBold"
    Instance.new("UICorner", btnAuto)

    btnAuto.MouseButton1Click:Connect(function()
        autoFarmActive = not autoFarmActive
        btnAuto.Text = autoFarmActive and "AUTO-FARM: ON" or "AUTO-FARM: OFF"
        btnAuto.BackgroundColor3 = autoFarmActive and Color3.fromRGB(0, 200, 80) or Color3.fromRGB(30, 30, 30)
    end)
end

-- [LOOPS]
task.spawn(function()
    while true do
        if autoFarmActive then
            pcall(function()
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj.Name == itemName and obj:IsA("BasePart") then
                        obj.CFrame = player.Character.HumanoidRootPart.CFrame
                    end
                end
                teleportarMoedas()
            end)
        end
        task.wait(0.3)
    end
end)

player.Idled:Connect(function()
    if antiAfkActive then
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end
end)

criarVermelhoHub()
criarRedzMenu()
