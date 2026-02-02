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

-- [ FUNÇÃO DE FARM ]
local function executarFarm(tipoBusca)
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") then
            local devePuxar = false
            local nomeBaixo = string.lower(obj.Name)

            if tipoBusca == "ALL_COINS" and string.find(nomeBaixo, "coin") then
                devePuxar = true
            elseif tipoBusca == "GOLD" and obj.Name == "GoldBar" then
                devePuxar = true
            elseif tipoBusca == "RADIOACTIVE" and obj.Name == "Radioactive Coin" then
                if obj:FindFirstAncestor("EventParts") or obj.Parent.Name == "Radioactive Coin" then
                    devePuxar = true
                end
            end

            if devePuxar then
                obj.CanCollide = false
                obj.Anchored = false 
                obj.CFrame = hrp.CFrame
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

    -- DEFINIÇÃO DAS ABAS
    local function abaEvento()
        clearContent()
        createToggle("Farm All Coins", function(v) farmAllCoinsActive = v end, farmAllCoinsActive)
        createToggle("Farm GoldBar", function(v) farmGoldActive = v end, farmGoldActive)
        createToggle("Farm Radioactive", function(v) farmRadioactiveActive = v end, farmRadioactiveActive)
    end

    local function abaMisc()
        clearContent()
        createToggle("Fly (Voo)", function(v) flyActive = v; if v then startFly() end end, flyActive)
        local speedInput = Instance.new("TextBox", content); speedInput.Size = UDim2.new(0.95, 0, 0, 35); speedInput.BackgroundColor3 = Color3.fromRGB(40, 40, 40); speedInput.Text = tostring(flySpeed); speedInput.TextColor3 = Color3.new(1, 1, 1); speedInput.PlaceholderText = "Velocidade Fly"; Instance.new("UICorner", speedInput)
        speedInput.FocusLost:Connect(function() flySpeed = tonumber(speedInput.Text) or 50 end)
        createToggle("Anti-AFK", function(v) antiAfkActive = v end, antiAfkActive)
    end

    local function abaFPS()
        clearContent()
        local b = Instance.new("TextButton", content); b.Size = UDim2.new(0.95, 0, 0, 40); b.Text = "Remover Texturas (Lag)"; b.BackgroundColor3 = Color3.fromRGB(30, 30, 30); b.TextColor3 = Color3.new(1, 1, 1); Instance.new("UICorner", b)
        b.MouseButton1Click:Connect(function() for _, v in pairs(game:GetDescendants()) do if v:IsA("BasePart") then v.Material = Enum.Material.SmoothPlastic elseif v:IsA("Decal") then v:Destroy() end end end)
    end

    -- ADICIONA OS BOTÕES NA SIDEBAR
    addTab("Evento", abaEvento)
    addTab("Misc", abaMisc)
    addTab("FPS", abaFPS)

    -- ABRE NA ABA EVENTO AUTOMATICAMENTE
    abaEvento()
end

-- [ LOOPS ]
task.spawn(function()
    while true do
        pcall(function()
            if farmAllCoinsActive then executarFarm("ALL_COINS") end
            if farmGoldActive then executarFarm("GOLD") end
            if farmRadioactiveActive then executarFarm("RADIOACTIVE") end
        end)
        task.wait(0.5)
    end
end)

player.Idled:Connect(function() if antiAfkActive then VirtualUser:CaptureController(); VirtualUser:ClickButton2(Vector2.new()) end end)

criarVermelhoHub()
