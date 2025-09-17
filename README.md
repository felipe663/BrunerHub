local Players = game:GetService("Players")
local player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

-- Variáveis globais
local flying, flySpeed = false, 50
local flyBodyVelocity = nil
local flyMultipliers = {0.5, 1, 2, 3}
local flyBaseSpeed = 50

local speedhackEnabled, speedMultiplier = false, 2
local noclipEnabled, noclipConnection = false, nil
local superJumpEnabled, jumpMultiplier, jumpConnection = false, 2.5, nil

-- Referências para os personagens atuais
local character = nil
local humanoid = nil
local rootPart = nil

-- ========== SISTEMA DE RESET AO MORRER ==========
local function setupCharacter()
    -- Remover conexões antigas
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    
    if jumpConnection then
        jumpConnection:Disconnect()
        jumpConnection = nil
    end
    
    -- Obter novo personagem
    character = player.Character
    if not character then
        character = player.CharacterAdded:Wait()
    end
    
    humanoid = character:WaitForChild("Humanoid")
    rootPart = character:WaitForChild("HumanoidRootPart")
    
    print("Personagem configurado! JumpPower atual: " .. humanoid.JumpPower)
    
    -- Reconectar funcionalidades se estiverem ativas
    if superJumpEnabled then
        humanoid.JumpPower = 50 * jumpMultiplier
        print("Super Jump reativado: " .. humanoid.JumpPower)
        
        jumpConnection = RunService.Heartbeat:Connect(function()
            if superJumpEnabled and humanoid and humanoid.Parent then
                humanoid.JumpPower = 50 * jumpMultiplier
            end
        end)
    end
    
    if noclipEnabled then
        noclipConnection = RunService.Stepped:Connect(function()
            if character and character:IsDescendantOf(workspace) then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
        print("Noclip reativado")
    end
    
    if speedhackEnabled then
        humanoid.WalkSpeed = 16 * speedMultiplier
        print("Speedhack reativado: " .. humanoid.WalkSpeed)
    end
    
    -- Conectar evento de morte
    humanoid.Died:Connect(function()
        print("Personagem morreu! Reiniciando em 2 segundos...")
        wait(2)
        setupCharacter()
    end)
end

-- Inicializar o personagem quando o jogo começar
player.CharacterAdded:Connect(function(char)
    wait(1)  -- Esperar um pouco para o personagem carregar completamente
    setupCharacter()
end)

-- Inicializar agora
if player.Character then
    setupCharacter()
end

-- ========== FUNÇÕES DE UTILIDADE ==========
local function toggleNoclip()
    noclipEnabled = not noclipEnabled
    
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    
    if noclipEnabled then
        noclipConnection = RunService.Stepped:Connect(function()
            if character and character:IsDescendantOf(workspace) then
                for _, part in pairs(character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end
        end)
        print("Noclip ATIVADO")
    else
        -- Restaurar colisão quando noclip é desativado
        if character and character:IsDescendantOf(workspace) then
            for _, part in pairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = true
                end
            end
        end
        print("Noclip DESATIVADO")
    end
end

local function toggleSuperJump()
    superJumpEnabled = not superJumpEnabled
    
    if superJumpEnabled then
        -- Ativar super pulo - 2.5x
        if humanoid then
            humanoid.JumpPower = 50 * jumpMultiplier
            print("Super Jump ATIVADO - JumpPower: " .. humanoid.JumpPower)
        end
        
        -- Forçar o valor periodicamente
        if jumpConnection then
            jumpConnection:Disconnect()
        end
        
        jumpConnection = RunService.Heartbeat:Connect(function()
            if superJumpEnabled and humanoid and humanoid.Parent then
                humanoid.JumpPower = 50 * jumpMultiplier
            end
        end)
    else
        -- Desativar super pulo
        if humanoid then
            humanoid.JumpPower = 50
            print("Super Jump DESATIVADO - JumpPower: " .. humanoid.JumpPower)
        end
        
        if jumpConnection then
            jumpConnection:Disconnect()
            jumpConnection = nil
        end
    end
end

-- ========== INTERFACE DE VELOCIDADE ==========
local speedGui
local function createSpeedInterface()
    if speedGui then speedGui:Destroy() end
    
    speedGui = Instance.new("ScreenGui", player.PlayerGui)
    speedGui.Name = "BrunerSpeedInterface"
    speedGui.ResetOnSpawn = false
    speedGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local speedContainer = Instance.new("Frame", speedGui)
    speedContainer.Name = "SpeedContainer"
    speedContainer.Size = UDim2.new(0, 200, 0, 140)
    speedContainer.Position = UDim2.new(0.68, 0, 0.32, 0)
    speedContainer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    speedContainer.BackgroundTransparency = 0.15
    speedContainer.BorderSizePixel = 0
    speedContainer.Active = true
    speedContainer.Draggable = true
    
    local uiCorner = Instance.new("UICorner", speedContainer)
    uiCorner.CornerRadius = UDim.new(0, 8)

    local titleBar = Instance.new("Frame", speedContainer)
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    titleBar.BorderSizePixel = 0
    
    local titleCorner = Instance.new("UICorner", titleBar)
    titleCorner.CornerRadius = UDim.new(0, 8)

    local titleLabel = Instance.new("TextLabel", titleBar)
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = "SPEEDHACK ⚡"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.BackgroundTransparency = 1

    local closeButton = Instance.new("TextButton", titleBar)
    closeButton.Size = UDim2.new(0, 30, 0, 25)
    closeButton.Position = UDim2.new(1, -35, 0, 2)
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 14
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    closeButton.BackgroundTransparency = 0.2
    closeButton.BorderSizePixel = 0
    
    local closeCorner = Instance.new("UICorner", closeButton)
    closeCorner.CornerRadius = UDim.new(0, 6)

    local speedToggle = Instance.new("TextButton", speedContainer)
    speedToggle.Size = UDim2.new(0, 160, 0, 35)
    speedToggle.Position = UDim2.new(0.5, -80, 0, 40)
    speedToggle.BackgroundColor3 = speedhackEnabled and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
    speedToggle.BackgroundTransparency = 0.2
    speedToggle.Text = speedhackEnabled and "DESATIVAR" or "ATIVAR"
    speedToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedToggle.TextSize = 14
    speedToggle.Font = Enum.Font.GothamBold
    speedToggle.BorderSizePixel = 0
    
    local toggleCorner = Instance.new("UICorner", speedToggle)
    toggleCorner.CornerRadius = UDim.new(0, 6)

    local speeds = {4, 5, 6, 7}
    local btns = {}
    
    for i, v in ipairs(speeds) do
        local btn = Instance.new("TextButton", speedContainer)
        btn.Size = UDim2.new(0, 35, 0, 25)
        btn.Position = UDim2.new(0, 15 + ((i-1)*45), 0, 90)
        btn.BackgroundColor3 = speedMultiplier == v and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
        btn.BackgroundTransparency = 0.2
        btn.Text = tostring(v).."x"
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextSize = 12
        btn.BorderSizePixel = 0
        btn.Font = Enum.Font.GothamBold
        
        local btnCorner = Instance.new("UICorner", btn)
        btnCorner.CornerRadius = UDim.new(0, 6)
        
        btn.MouseButton1Click:Connect(function()
            speedMultiplier = v
            for j, button in ipairs(btns) do
                button.BackgroundColor3 = (speeds[j] == v) and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
            end
            if speedhackEnabled and humanoid then
                humanoid.WalkSpeed = 16 * speedMultiplier
            end
        end)
        table.insert(btns, btn)
    end

    speedToggle.MouseButton1Click:Connect(function()
        speedhackEnabled = not speedhackEnabled
        speedToggle.Text = speedhackEnabled and "DESATIVAR" or "ATIVAR"
        speedToggle.BackgroundColor3 = speedhackEnabled and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
        
        if speedhackEnabled and humanoid then
            humanoid.WalkSpeed = 16 * speedMultiplier
        elseif humanoid then
            humanoid.WalkSpeed = 16
        end
    end)

    closeButton.MouseButton1Click:Connect(function()
        speedGui:Destroy()
        if speedhackEnabled and humanoid then
            humanoid.WalkSpeed = 16
            speedhackEnabled = false
        end
    end)
end

-- ========== INTERFACE DE VOO ==========
local flyGui
local function createFlyInterface()
    if flyGui then flyGui:Destroy() end
    
    flyGui = Instance.new("ScreenGui", player.PlayerGui)
    flyGui.Name = "BrunerFlyInterface"
    flyGui.ResetOnSpawn = false
    flyGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local flyContainer = Instance.new("Frame", flyGui)
    flyContainer.Name = "FlyContainer"
    flyContainer.Size = UDim2.new(0, 220, 0, 160)
    flyContainer.Position = UDim2.new(0.62, 0, 0.19, 0)
    flyContainer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    flyContainer.BackgroundTransparency = 0.15
    flyContainer.BorderSizePixel = 0
    flyContainer.Active = true
    flyContainer.Draggable = true
    
    local uiCorner = Instance.new("UICorner", flyContainer)
    uiCorner.CornerRadius = UDim.new(0, 8)

    local titleBar = Instance.new("Frame", flyContainer)
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    titleBar.BorderSizePixel = 0
    
    local titleCorner = Instance.new("UICorner", titleBar)
    titleCorner.CornerRadius = UDim.new(0, 8)

    local titleLabel = Instance.new("TextLabel", titleBar)
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = "FLY 🚀"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.BackgroundTransparency = 1

    local closeButton = Instance.new("TextButton", titleBar)
    closeButton.Size = UDim2.new(0, 30, 0, 25)
    closeButton.Position = UDim2.new(1, -35, 0, 2)
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 14
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    closeButton.BackgroundTransparency = 0.2
    closeButton.BorderSizePixel = 0
    
    local closeCorner = Instance.new("UICorner", closeButton)
    closeCorner.CornerRadius = UDim.new(0, 6)

    local flyToggle = Instance.new("TextButton", flyContainer)
    flyToggle.Size = UDim2.new(0, 180, 0, 35)
    flyToggle.Position = UDim2.new(0.5, -90, 0, 40)
    flyToggle.BackgroundColor3 = flying and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
    flyToggle.BackgroundTransparency = 0.2
    flyToggle.Text = flying and "DESATIVAR FLY" or "ATIVAR FLY"
    flyToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    flyToggle.TextSize = 14
    flyToggle.Font = Enum.Font.GothamBold
    flyToggle.BorderSizePixel = 0
    
    local toggleCorner = Instance.new("UICorner", flyToggle)
    toggleCorner.CornerRadius = UDim.new(0, 6)

    local speedLabel = Instance.new("TextLabel", flyContainer)
    speedLabel.Size = UDim2.new(1, 0, 0, 20)
    speedLabel.Position = UDim2.new(0, 0, 0, 85)
    speedLabel.Text = "VELOCIDADE:"
    speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedLabel.TextSize = 12
    speedLabel.Font = Enum.Font.GothamBold
    speedLabel.BackgroundTransparency = 1

    local flyBtns = {}
    local currentFlyMultiplier = 1

    for i, mult in ipairs(flyMultipliers) do
        local btn = Instance.new("TextButton", flyContainer)
        btn.Size = UDim2.new(0, 40, 0, 25)
        btn.Position = UDim2.new(0, 15 + ((i-1)*50), 0, 110)
        btn.Text = tostring(mult).."x"
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextSize = 12
        btn.Font = Enum.Font.GothamBold
        btn.BackgroundColor3 = currentFlyMultiplier == mult and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
        btn.BackgroundTransparency = 0.2
        btn.BorderSizePixel = 0
        
        local btnCorner = Instance.new("UICorner", btn)
        btnCorner.CornerRadius = UDim.new(0, 6)
        
        btn.MouseButton1Click:Connect(function()
            currentFlyMultiplier = mult
            flySpeed = flyBaseSpeed * currentFlyMultiplier
            for j, b in ipairs(flyBtns) do
                b.BackgroundColor3 = flyMultipliers[j] == currentFlyMultiplier and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
            end
            if flying and flyBodyVelocity then
                flyBodyVelocity.Velocity = workspace.CurrentCamera.CFrame.LookVector * flySpeed
            end
        end)
        table.insert(flyBtns, btn)
    end

    flyToggle.MouseButton1Click:Connect(function()
        flying = not flying
        flyToggle.Text = flying and "DESATIVAR FLY" or "ATIVAR FLY"
        flyToggle.BackgroundColor3 = flying and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
        
        if flying and rootPart then
            flyBodyVelocity = Instance.new("BodyVelocity", rootPart)
            flyBodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
            flyBodyVelocity.Velocity = workspace.CurrentCamera.CFrame.LookVector * flySpeed
            humanoid.PlatformStand = false
        else
            if flyBodyVelocity then flyBodyVelocity:Destroy() end
            flyBodyVelocity = nil
        end
    end)

    local flyConnection
    flyConnection = RunService.Heartbeat:Connect(function()
        if flying and flyBodyVelocity then
            flyBodyVelocity.Velocity = workspace.CurrentCamera.CFrame.LookVector * flySpeed
        end
    end)

    closeButton.MouseButton1Click:Connect(function()
        flyGui:Destroy()
        flying = false
        if flyBodyVelocity then flyBodyVelocity:Destroy() end
        flyBodyVelocity = nil
        if flyConnection then flyConnection:Disconnect() end
    end)
end

-- ========== INTERFACE DE TELEPORTE ==========
local teleportGui
local function createTeleportInterface()
    if teleportGui then teleportGui:Destroy() end
    
    teleportGui = Instance.new("ScreenGui", player.PlayerGui)
    teleportGui.Name = "BrunerTeleportInterface"
    teleportGui.ResetOnSpawn = false
    teleportGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local mainFrame = Instance.new("Frame", teleportGui)
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 400, 0, 300)
    mainFrame.Position = UDim2.new(0.42, 0, 0.24, 0)
    mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    mainFrame.BackgroundTransparency = 0.15
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    
    local uiCorner = Instance.new("UICorner", mainFrame)
    uiCorner.CornerRadius = UDim.new(0, 8)

    local titleBar = Instance.new("Frame", mainFrame)
    titleBar.Size = UDim2.new(1, 0, 0, 35)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    titleBar.BorderSizePixel = 0
    
    local titleCorner = Instance.new("UICorner", titleBar)
    titleCorner.CornerRadius = UDim.new(0, 8)

    local titleLabel = Instance.new("TextLabel", titleBar)
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = "TELEPORTAR PARA PLAYER"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 14
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.BackgroundTransparency = 1

    local closeButton = Instance.new("TextButton", titleBar)
    closeButton.Size = UDim2.new(0, 30, 0, 25)
    closeButton.Position = UDim2.new(1, -35, 0, 5)
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 14
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    closeButton.BackgroundTransparency = 0.2
    closeButton.BorderSizePixel = 0
    
    local closeCorner = Instance.new("UICorner", closeButton)
    closeCorner.CornerRadius = UDim.new(0, 6)

    local refreshButton = Instance.new("TextButton", mainFrame)
    refreshButton.Size = UDim2.new(0, 120, 0, 35)
    refreshButton.Position = UDim2.new(0.5, -60, 0, 45)
    refreshButton.Text = "ATUALIZAR 🔄"
    refreshButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    refreshButton.TextSize = 14
    refreshButton.Font = Enum.Font.GothamBold
    refreshButton.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    refreshButton.BackgroundTransparency = 0.2
    refreshButton.BorderSizePixel = 0
    
    local refreshCorner = Instance.new("UICorner", refreshButton)
    refreshCorner.CornerRadius = UDim.new(0, 6)

    local menuFrame = Instance.new("Frame", mainFrame)
    menuFrame.Name = "MenuFrame"
    menuFrame.Size = UDim2.new(0, 370, 0, 150)
    menuFrame.Position = UDim2.new(0.5, -185, 0, 90)
    menuFrame.BackgroundTransparency = 1
    menuFrame.BorderSizePixel = 0

    local scrollPlayers = Instance.new("ScrollingFrame", menuFrame)
    scrollPlayers.Size = UDim2.new(0.7, 0, 1, 0)
    scrollPlayers.Position = UDim2.new(0, 0, 0, 0)
    scrollPlayers.BackgroundTransparency = 1
    scrollPlayers.BorderSizePixel = 0
    scrollPlayers.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollPlayers.ScrollBarThickness = 6

    local layout = Instance.new("UIListLayout", scrollPlayers)
    layout.SortOrder = Enum.SortOrder.LayoutOrder
    layout.Padding = UDim.new(0, 8)

    local selectedPlayer = nil

    -- Botão de teleporte ao lado da lista
    local teleportBtn = Instance.new("TextButton", menuFrame)
    teleportBtn.Size = UDim2.new(0, 100, 0, 40)
    teleportBtn.Position = UDim2.new(0.75, 10, 0.5, -20)
    teleportBtn.Text = "TELEPORTAR"
    teleportBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    teleportBtn.TextSize = 14
    teleportBtn.Font = Enum.Font.GothamBold
    teleportBtn.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    teleportBtn.BackgroundTransparency = 0.2
    teleportBtn.BorderSizePixel = 0
    
    local teleportCorner = Instance.new("UICorner", teleportBtn)
    teleportCorner.CornerRadius = UDim.new(0, 8)

    local function refreshPlayers()
        for _, c in ipairs(scrollPlayers:GetChildren()) do
            if c:IsA("TextButton") then c:Destroy() end
        end
        
        local count = 0
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player then
                count = count + 1
                local btn = Instance.new("TextButton", scrollPlayers)
                btn.Size = UDim2.new(1, 0, 0, 35)
                btn.Text = p.DisplayName .. " ("..p.Name..")"
                btn.TextColor3 = Color3.fromRGB(255, 255, 255)
                btn.TextSize = 12
                btn.Font = Enum.Font.GothamBold
                btn.BackgroundColor3 = selectedPlayer == p and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
                btn.BackgroundTransparency = 0.2
                btn.BorderSizePixel = 0
                
                local btnCorner = Instance.new("UICorner", btn)
                btnCorner.CornerRadius = UDim.new(0, 6)
                
                btn.MouseButton1Click:Connect(function()
                    selectedPlayer = p
                    for _, b in ipairs(scrollPlayers:GetChildren()) do
                        if b:IsA("TextButton") then
                            b.BackgroundColor3 = b == btn and Color3.fromRGB(0, 128, 0) or Color3.fromRGB(75, 0, 130)
                        end
                    end
                end)
            end
        end
        scrollPlayers.CanvasSize = UDim2.new(0, 0, 0, count * 43)
    end

    refreshButton.MouseButton1Click:Connect(refreshPlayers)
    refreshPlayers()

    teleportBtn.MouseButton1Click:Connect(function()
        if selectedPlayer and selectedPlayer.Character and selectedPlayer.Character:FindFirstChild("HumanoidRootPart") and rootPart then
            rootPart.CFrame = selectedPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
        end
    end)

    closeButton.MouseButton1Click:Connect(function()
        teleportGui:Destroy()
    end)
end

-- ========== INTERFACE PRINCIPAL ==========
local mainGui
local function createMainInterface()
    if mainGui then mainGui:Destroy() end
    
    mainGui = Instance.new("ScreenGui", player.PlayerGui)
    mainGui.Name = "BrunerMainInterface"
    mainGui.ResetOnSpawn = false
    mainGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local mainContainer = Instance.new("Frame", mainGui)
    mainContainer.Name = "MainContainer"
    mainContainer.Size = UDim2.new(0, 200, 0, 250)
    mainContainer.Position = UDim2.new(0.12, 0, 0.32, 0)
    mainContainer.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
    mainContainer.BackgroundTransparency = 0.15
    mainContainer.BorderSizePixel = 0
    mainContainer.Active = true
    mainContainer.Draggable = true
    
    local uiCorner = Instance.new("UICorner", mainContainer)
    uiCorner.CornerRadius = UDim.new(0, 8)

    local titleBar = Instance.new("Frame", mainContainer)
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    titleBar.BorderSizePixel = 0
    
    local titleCorner = Instance.new("UICorner", titleBar)
    titleCorner.CornerRadius = UDim.new(0, 8)

    local titleLabel = Instance.new("TextLabel", titleBar)
    titleLabel.Size = UDim2.new(1, -40, 1, 0)
    titleLabel.Position = UDim2.new(0, 10, 0, 0)
    titleLabel.Text = "BRUNER HUB ⚡"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.BackgroundTransparency = 1

    local closeButton = Instance.new("TextButton", titleBar)
    closeButton.Size = UDim2.new(0, 30, 0, 25)
    closeButton.Position = UDim2.new(1, -35, 0, 2)
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.TextSize = 14
    closeButton.Font = Enum.Font.GothamBold
    closeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    closeButton.BackgroundTransparency = 0.2
    closeButton.BorderSizePixel = 0
    
    local closeCorner = Instance.new("UICorner", closeButton)
    closeCorner.CornerRadius = UDim.new(0, 6)

    -- Funções na interface principal
    local buttons = {
        {name = "NOCLIP", func = toggleNoclip, state = noclipEnabled},
        {name = "SUPER JUMP", func = toggleSuperJump, state = superJumpEnabled},
        {name = "SPEEDHACK ⚡", func = createSpeedInterface, state = false},
        {name = "FLY 🚀", func = createFlyInterface, state = false},
        {name = "TELEPORT ⚡", func = createTeleportInterface, state = false}
    }

    local buttonHeight = 35
    local buttonSpacing = 10
    local startPosition = 40

    for i, buttonInfo in ipairs(buttons) do
        local buttonFrame = Instance.new("Frame", mainContainer)
        buttonFrame.Size = UDim2.new(0, 180, 0, buttonHeight)
        buttonFrame.Position = UDim2.new(0, 10, 0, startPosition + (i-1)*(buttonHeight + buttonSpacing))
        buttonFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        buttonFrame.BackgroundTransparency = 0.3
        buttonFrame.BorderSizePixel = 0
        
        local buttonCorner = Instance.new("UICorner", buttonFrame)
        buttonCorner.CornerRadius = UDim.new(0, 6)

        local button = Instance.new("TextButton", buttonFrame)
        button.Size = UDim2.new(1, 0, 1, 0)
        button.Position = UDim2.new(0, 0, 0, 0)
        button.Text = buttonInfo.name
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.TextSize = 12
        button.Font = Enum.Font.GothamBold
        button.BackgroundTransparency = 1
        button.BorderSizePixel = 0
        button.ZIndex = 2

        -- Indicador visual de estado (apenas para NOCLIP e SUPER JUMP)
        if buttonInfo.func == toggleNoclip or buttonInfo.func == toggleSuperJump then
            local statusIndicator = Instance.new("Frame", buttonFrame)
            statusIndicator.Size = UDim2.new(0, 10, 0, 10)
            statusIndicator.Position = UDim2.new(1, -15, 0.5, -5)
            statusIndicator.BackgroundColor3 = buttonInfo.state and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
            statusIndicator.BorderSizePixel = 0
            
            local indicatorCorner = Instance.new("UICorner", statusIndicator)
            indicatorCorner.CornerRadius = UDim.new(1, 0)
        end

        button.MouseButton1Click:Connect(function()
            buttonInfo.func()
            -- Atualizar o estado visual para NOCLIP e SUPER JUMP
            if buttonInfo.func == toggleNoclip or buttonInfo.func == toggleSuperJump then
                buttonInfo.state = not buttonInfo.state
                -- Recriar a interface para atualizar o indicador visual
                createMainInterface()
            end
        end)
    end

    closeButton.MouseButton1Click:Connect(function()
        mainGui:Destroy()
    end)
end

-- ========== BOTÃO FLUTUANTE PARA ABRIR A INTERFACE ==========
local floatingButton
local function createFloatingButton()
    if floatingButton then floatingButton:Destroy() end
    
    floatingButton = Instance.new("ScreenGui", player.PlayerGui)
    floatingButton.Name = "BrunerFloatingButton"
    floatingButton.ResetOnSpawn = false
    floatingButton.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

    local button = Instance.new("TextButton", floatingButton)
    button.Size = UDim2.new(0, 70, 0, 70)
    button.Position = UDim2.new(0, 20, 0.5, -35)
    button.BackgroundColor3 = Color3.fromRGB(75, 0, 130)
    button.BackgroundTransparency = 0.2
    button.Text = "BRUNER\nHUB"
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 14
    button.Font = Enum.Font.Gotham
    button.TextWrapped = true
    button.BorderSizePixel = 0
    button.Active = true
    button.Draggable = true
    
    local buttonCorner = Instance.new("UICorner", button)
    buttonCorner.CornerRadius = UDim.new(1, 0)
    
    local uiStroke = Instance.new("UIStroke", button)
    uiStroke.Color = Color3.fromRGB(255, 255, 255)
    uiStroke.Thickness = 2
    
    -- Efeito de brilho ao passar o mouse
    button.MouseEnter:Connect(function()
        game:GetService("TweenService"):Create(button, TweenInfo.new(0.2), {BackgroundTransparency = 0}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        game:GetService("TweenService"):Create(button, TweenInfo.new(0.2), {BackgroundTransparency = 0.2}):Play()
    end)
    
    button.MouseButton1Click:Connect(function()
        createMainInterface()
    end)
    
    return button
end

-- ========== INICIAR BOTÃO FLUTUANTE ==========
local floatingBtn = createFloatingButton()

-- Verificar se a interface principal foi fechada para garantir que o botão fique sempre visível
local function checkMainInterface()
    while true do
        wait(1)
        if not mainGui or not mainGui.Parent then
            if floatingBtn then
                floatingBtn.Visible = true
            else
                floatingBtn = createFloatingButton()
            end
        end
    end
end

-- Iniciar a verificação em uma thread separada
spawn(checkMainInterface)
