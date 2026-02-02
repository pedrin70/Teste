local Players = game:GetService("Players")
local player = Players.LocalPlayer
local runService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

-- [ VARIÁVEIS DE CONTROLE ]
local farmAllCoinsActive = false
local farmGoldActive = false
local farmRadioactiveActive = false 
local antiAfkActive = false
local flyActive = false
local flySpeed = 50 

-- [ FUNÇÃO DE VOO ]
local function startFly()
    local char = player.Character or player.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local hum = char:WaitForChild("Humanoid")
    for _, v in pairs(hrp:GetChildren()) do if v.Name == "VooVel" or v.Name == "VooGiro" then v:Destroy() end end
    local velocity = Instance.new("BodyVelocity")
    velocity.Name = "VooVel"; velocity.Parent = hrp; velocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    local gyro = Instance.new("BodyGyro")
    gyro.Name = "VooGiro"; gyro.Parent = hrp; gyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
    hum.PlatformStand = true 
    task.spawn(function()
        while flyActive and char.Parent and hum.Health > 0 do
            local camera = workspace.CurrentCamera
            velocity.Velocity = hum.MoveDirection.Magnitude > 0 and camera.CFrame.LookVector * flySpeed or Vector3.new(0, 0, 0)
            gyro.CFrame = camera.CFrame
            runService.Heartbeat:Wait()
        end
        if velocity then velocity:Destroy() end if gyro then gyro:Destroy() end if hum then hum.PlatformStand = false end
    end)
end

-- [ FUNÇÃO DE FARM - SISTEMA IGUAL AO RADIOACTIVE ]
local function executarFarm()
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    for _, obj in pairs(workspace:GetDescendants()) do
        local devePuxar = false
        
        -- SISTEMA GOLD (IGUAL AO RADIOACTIVE)
        if farmGoldActive and obj.Name == "Main" then
            -- Verifica se o pai é GoldBar e se está na pasta MoneyEventParts (como na foto)
            if obj.Parent.Name == "GoldBar" and obj:FindFirstAncestor("MoneyEventParts") then
                devePuxar = true
            end
        
        -- SISTEMA RADIOACTIVE
        elseif farmRadioactiveActive and obj.Name == "Radioactive Coin" then
            if obj:FindFirstAncestor("EventParts") or obj.Parent.Name == "Radioactive Coin" then
                devePuxar = true
            end

        -- SISTEMA ALL COINS
        elseif farmAllCoinsActive and string.find(string.lower(obj.Name), "coin") then
            devePuxar = true
        end

        -- EXECUÇÃO DA COLETA
        if devePuxar and obj:IsA("BasePart") then
            obj.CanCollide = false
            obj.CFrame = hrp.CFrame
            if firetouchinterest then
                firetouchinterest(hrp, obj, 0)
                firetouchinterest(hrp, obj, 1)
            end
        end
    end
end

-- [ INTERFACE ]
local function criarVermelhoHub()
    if player.PlayerGui:FindFirstChild("VermelhoHubGui") then player.PlayerGui.VermelhoHubGui:Destroy() end
    local sg = Instance.new("ScreenGui", player.PlayerGui); sg.Name = "VermelhoHubGui"; sg.ResetOnSpawn = false
    
    local minBtn = Instance.new("TextButton", sg); minBtn.Size = UDim2.new(0, 50, 0, 50); minBtn.Position = UDim2.new(0, 10, 0.5, -25); minBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0); minBtn.Text = "V"; minBtn.TextColor3 = Color3.fromRGB(255, 0, 0); minBtn.Font = "GothamBold"; minBtn.TextSize = 25; minBtn.Active = true; minBtn.Draggable = true; Instance.new("UICorner", minBtn).CornerRadius = UDim.new(1, 0)

    local main = Instance.new("Frame", sg); main.Size = UDim2.new(0, 420, 0, 280); main.Position = UDim2.new(0.5, -210, 0.5, -140); main.BackgroundColor3 = Color3.fromRGB(15, 15, 15); main.Active = true; main.Draggable = true; Instance.new("UICorner", main)
    minBtn.MouseButton1Click:Connect(function() main.Visible = not main.Visible end)

    local sidebar = Instance.new("Frame", main); sidebar.Size = UDim2.new(0, 100, 1, -35); sidebar.BackgroundColor3 = Color3.fromRGB(20, 20, 20); Instance.new("UICorner", sidebar); Instance.new("UIListLayout", sidebar).HorizontalAlignment = "Center"
    local content = Instance.new("ScrollingFrame", main); content.Position = UDim2.new(0, 110, 0, 10); content.Size = UDim2.new(1, -120, 1, -50); content.BackgroundTransparency = 1; content.ScrollBarThickness = 2; Instance.new("UIListLayout", content).Padding = UDim.new(0, 10)

    local function clearContent() for _, v in pairs(content:GetChildren()) do if not v:IsA("UIListLayout") then v:Destroy() end end end
    
    local function createToggle(text, callback, state)
        local btn = Instance.new("TextButton", content); btn.Size = UDim2.new(0.95, 0, 0, 40); btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35); btn.Text = text .. (state and ": ON" or ": OFF"); btn.TextColor3 = Color3.new(1, 1, 1); btn.Font = "Gotham"; Instance.new("UICorner", btn)
        btn.MouseButton1Click:Connect(function()
            state = not state; btn.Text = text .. (state and ": ON" or ": OFF"); btn.BackgroundColor3 = state and Color3.fromRGB(200, 0, 0) or Color3.fromRGB(35, 35, 35); callback(state)
        end)
    end

    local function addTab(name, func)
        local b = Instance.new("TextButton", sidebar); b.Size = UDim2.new(0.9, 0, 0, 35); b.Text = name; b.BackgroundColor3 = Color3.fromRGB(30, 30, 30); b.TextColor3 = Color3.new(1, 1, 1); b.Font = "GothamBold"; Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function() clearContent(); func() end)
    end

    local function abaEvento()
        clearContent()
        createToggle("Farm UFO Coins(fase de testes)", function(v) farmAllCoinsActive = v end, farmAllCoinsActive)
        createToggle("Farm GoldBar (beta)", function(v) farmGoldActive = v end, farmGoldActive)
        createToggle("Farm Radioactive(fase de testes)", function(v) farmRadioactiveActive = v end, farmRadioactiveActive)
    end

    addTab("Evento", abaEvento)
    addTab("Misc", function()
        clearContent()
        createToggle("Fly (Voo)", function(v) flyActive = v; if v then startFly() end end, flyActive)
        createToggle("Anti-AFK", function(v) antiAfkActive = v end, antiAfkActive)
    end)

    addTab("FPS", function()
        clearContent()
        local b = Instance.new("TextButton", content); b.Size = UDim2.new(0.95, 0, 0, 40); b.Text = "Remover Texturas"; b.BackgroundColor3 = Color3.fromRGB(30, 30, 30); b.TextColor3 = Color3.new(1, 1, 1); Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function() 
            for _, v in pairs(game:GetDescendants()) do if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic elseif v:IsA("Decal") then v:Destroy() end end 
        end)
    end)

    abaEvento()
end

-- [ LOOP ]
task.spawn(function()
    while true do
        pcall(executarFarm)
        task.wait(0.3)
    end
end)

player.Idled:Connect(function() if antiAfkActive then VirtualUser:CaptureController(); VirtualUser:ClickButton2(Vector2.new()) end end)
criarVermelhoHub()
