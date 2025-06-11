local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local Player = Players.LocalPlayer
local Mouse = Player:GetMouse()
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

-- Whitelist feita por mim XD
local Whitelist = {
    8220966839, 
    8026202614, 
    
}

-- Verifica se o jogador está na whitelist
local function isWhitelisted()
    for _, userId in ipairs(Whitelist) do
        if Player.UserId == userId then
            return true
        end
    end
    return false
end

-- Checa a whitelist e kicka se não estiver nela
if not isWhitelisted() then
    Player:Kick("Você não está na whitelist! Contate o administrador para obter acesso.")
    return -- Interrompe a execução do script
end

-- Prossegue com o menu se o jogador estiver na whitelist
local Window = Rayfield:CreateWindow({
    Name = "RGZIN MENU☠",
    LoadingTitle = "Carregando RGZIN MENU...",
    LoadingSubtitle = "By Seila and DiegoGG123890",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = nil,
        FileName = "MeuConfig"
    },
    Discord = {
        Enabled = false,
        Invite = "",
        RememberJoins = true
    },
    KeySystem = false
})

-- Abas
local TabJogador = Window:CreateTab("Jogador")
local TabPVP = Window:CreateTab("PVP")
local TabVisual = Window:CreateTab("Visual")

-- ===== FREECAM =====
local freecamEnabled = false
local freecamSpeed = 3
local freecamPos = Vector3.new()
local freecamRot = Vector2.new()
local camRotStart = Vector2.new()
local camPosStart = Vector3.new()
local inputConnections = {}

local moveVec = Vector3.new(0,0,0)

local function enableFreecam()
    freecamEnabled = true

    camRotStart = Vector2.new(Camera.CFrame:ToEulerAnglesYXZ())
    camPosStart = Camera.CFrame.Position
    freecamPos = camPosStart
    freecamRot = Vector2.new(camRotStart.X, camRotStart.Y)

    Camera.CameraType = Enum.CameraType.Scriptable

    UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
    UserInputService.MouseIconEnabled = false

    -- Congelar personagem (sem andar)
    Humanoid.WalkSpeed = 0
    Humanoid.JumpPower = 0

    moveVec = Vector3.new(0,0,0) -- reset movimento pra evitar bug do "andar sozinho"

    local function onInputBegan(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.W then
                moveVec = moveVec + Vector3.new(0,0,-1)
            elseif input.KeyCode == Enum.KeyCode.S then
                moveVec = moveVec + Vector3.new(0,0,1)
            elseif input.KeyCode == Enum.KeyCode.A then
                moveVec = moveVec + Vector3.new(-1,0,0)
            elseif input.KeyCode == Enum.KeyCode.D then
                moveVec = moveVec + Vector3.new(1,0,0)
            elseif input.KeyCode == Enum.KeyCode.E then
                moveVec = moveVec + Vector3.new(0,1,0)
            elseif input.KeyCode == Enum.KeyCode.Q then
                moveVec = moveVec + Vector3.new(0,-1,0)
            end
        end
    end

    local function onInputEnded(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            if input.KeyCode == Enum.KeyCode.W then
                moveVec = moveVec - Vector3.new(0,0,-1)
            elseif input.KeyCode == Enum.KeyCode.S then
                moveVec = moveVec - Vector3.new(0,0,1)
            elseif input.KeyCode == Enum.KeyCode.A then
                moveVec = moveVec - Vector3.new(-1,0,0)
            elseif input.KeyCode == Enum.KeyCode.D then
                moveVec = moveVec - Vector3.new(1,0,0)
            elseif input.KeyCode == Enum.KeyCode.E then
                moveVec = moveVec - Vector3.new(0,1,0)
            elseif input.KeyCode == Enum.KeyCode.Q then
                moveVec = moveVec - Vector3.new(0,-1,0)
            end
        end
    end

    table.insert(inputConnections, UserInputService.InputBegan:Connect(onInputBegan))
    table.insert(inputConnections, UserInputService.InputEnded:Connect(onInputEnded))

    table.insert(inputConnections, RunService.RenderStepped:Connect(function(deltaTime)
        local delta = UserInputService:GetMouseDelta()
        freecamRot = freecamRot + Vector2.new(-delta.Y, -delta.X) * 0.002
        freecamRot = Vector2.new(math.clamp(freecamRot.X, -math.pi/2, math.pi/2), freecamRot.Y)

        local cfRotation = CFrame.Angles(freecamRot.X, freecamRot.Y, 0)
        local moveDelta = moveVec * freecamSpeed

        freecamPos = freecamPos + (cfRotation * moveDelta) * deltaTime * 50
        Camera.CFrame = CFrame.new(freecamPos) * cfRotation
    end))
end

local function disableFreecam()
    freecamEnabled = false

    for _, conn in pairs(inputConnections) do
        conn:Disconnect()
    end
    inputConnections = {}

    Camera.CameraType = Enum.CameraType.Custom
    UserInputService.MouseBehavior = Enum.MouseBehavior.Default
    UserInputService.MouseIconEnabled = true

    -- Voltar a andar normalmente
    Humanoid.WalkSpeed = 16
    Humanoid.JumpPower = 50
end

TabJogador:CreateToggle({
    Name = "Freecam",
    CurrentValue = false,
    Flag = "FreecamToggle",
    Callback = function(value)
        if value then
            enableFreecam()
        else
            disableFreecam()
        end
    end
})

-- ===== Noclip =====
local noclipEnabled = false
local noclipConnection = nil

local function noclipOn()
    noclipEnabled = true
    noclipConnection = RunService.Stepped:Connect(function()
        if Character then
            for _, part in pairs(Character:GetChildren()) do
                if part:IsA("BasePart") and part.CanCollide == true then
                    part.CanCollide = false
                end
            end
        end
    end)
end

local function noclipOff()
    noclipEnabled = false
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    -- Deixar os CanCollide true? Pode ajustar aqui se quiser
end

TabJogador:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "NoclipToggle",
    Callback = function(value)
        if value then
            noclipOn()
        else
            noclipOff()
        end
    end
})

-- ===== Fly (executar ação) =====
TabJogador:CreateButton({
    Name = "FLY",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/XNEOFF/FlyGuiV3/main/FlyGuiV3.txt"))()
    end
})

-- ===== AIMBOT =====
local aiming = false
local target = nil

local maxDistance = 1000
local fovAngle = 90

local fovRadius = math.tan(math.rad(fovAngle / 2)) * Camera.ViewportSize.X / 2
local fovCircle = Drawing.new("Circle")
fovCircle.Color = Color3.fromRGB(0, 255, 0)
fovCircle.Thickness = 2
fovCircle.Transparency = 0.5
fovCircle.Filled = false
fovCircle.Radius = fovRadius
fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
fovCircle.Visible = false

local checkTeam = true -- para toggle "Não travar mira no time amigo"

local function isInFOV(targetPos)
    local camForward = Camera.CFrame.LookVector
    local directionToTarget = (targetPos - Camera.CFrame.Position).Unit
    local angle = math.deg(math.acos(camForward:Dot(directionToTarget)))
    return angle <= (fovAngle / 2)
end

local function getClosestEnemyInFOV()
    local closest = nil
    local shortestDistance = maxDistance

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= Player and player.Character then
            if not checkTeam or (player.Team ~= Player.Team) then
                local head = player.Character:FindFirstChild("Head")
                if head then
                    local pos = head.Position
                    local distance = (Camera.CFrame.Position - pos).Magnitude

                    if distance < shortestDistance and isInFOV(pos) then
                        shortestDistance = distance
                        closest = player
                    end
                end
            end
        end
    end

    return closest
end

local function aimAtTarget()
    if not target or not target.Character then return end
    local head = target.Character:FindFirstChild("Head")
    if head then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
    end
end

local aimingConnection = nil
local inputBeganConnection = nil
local inputEndedConnection = nil
local renderConnection = nil

local function enableAimbot()
    aimingConnection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = true
            fovCircle.Visible = true
        end
    end)

    inputEndedConnection = UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = false
            target = nil
            fovCircle.Visible = false
        end
    end)

    renderConnection = RunService.RenderStepped:Connect(function()
        fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        fovRadius = math.tan(math.rad(fovAngle / 2)) * Camera.ViewportSize.X / 2
        fovCircle.Radius = fovRadius

        if aiming then
            target = getClosestEnemyInFOV()
            if target then
                aimAtTarget()
            end
        end
    end)
end

local function disableAimbot()
    if aimingConnection then aimingConnection:Disconnect() end
    if inputEndedConnection then inputEndedConnection:Disconnect() end
    if renderConnection then renderConnection:Disconnect() end
    aiming = false
    target = nil
    fovCircle.Visible = false
end

TabPVP:CreateToggle({
    Name = "Ativar Aimbot (botão direito do mouse)",
    CurrentValue = false,
    Flag = "AimbotToggle",
    Callback = function(value)
        if value then
            enableAimbot()
        else
            disableAimbot()
        end
    end
})

TabPVP:CreateToggle({
    Name = "Não travar mira no time amigo",
    CurrentValue = true,
    Flag = "IgnoreTeamToggle",
    Callback = function(value)
        checkTeam = value
    end
})

-- Novo toggle para ajustar o FOV do aimbot
TabPVP:CreateSlider({
    Name = "Aimbot FOV",
    Range = {10, 180},
    Increment = 5,
    Suffix = "°",
    CurrentValue = 90,
    Flag = "AimbotFOV",
    Callback = function(value)
        fovAngle = value
        fovRadius = math.tan(math.rad(fovAngle / 2)) * Camera.ViewportSize.X / 2
        fovCircle.Radius = fovRadius
    end
})

-- ===== ESP =====
local ESP = {}
local espEnabled = false

local teamColors = {
    ["Ballas"] = Color3.fromRGB(128, 0, 128), -- Roxo
    ["Groove"] = Color3.fromRGB(0, 255, 0), -- Verde
    ["Staff"] = Color3.fromRGB(255, 255, 0), -- Amarelo
    ["Choque"] = Color3.fromRGB(0, 0, 0), -- Preto
    ["Civil"] = Color3.fromRGB(128, 128, 128), -- Cinza
    ["Holanda"] = Color3.fromRGB(255, 165, 0), -- Laranja
    ["França"] = Color3.fromRGB(0, 0, 139), -- Azul escuro
    ["Polícia Militar"] = Color3.fromRGB(135, 206, 250), -- Azul claro
    ["Vanilla"] = Color3.fromRGB(255, 192, 203), -- Rosa
}

local function createESP(player)
    local espBox = Drawing.new("Quad")
    espBox.Color = Color3.new(1,1,1)
    espBox.Thickness = 2
    espBox.Filled = false
    espBox.Visible = true

    return espBox
end

local function updateESP()
    for player, box in pairs(ESP) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Head") then
            local root = player.Character.HumanoidRootPart
            local head = player.Character.Head

            local rootPos, rootVis = Camera:WorldToViewportPoint(root.Position)
            local headPos, headVis = Camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0))

            if rootVis and headVis then
                -- Posições dos cantos do quadrado ESP (contorno mais correto)
                local sizeY = math.abs(headPos.Y - rootPos.Y)
                local sizeX = sizeY / 2

                local points = {
                    Vector2.new(rootPos.X - sizeX/2, rootPos.Y),          -- inferior esquerdo
                    Vector2.new(rootPos.X - sizeX/2, rootPos.Y - sizeY),  -- superior esquerdo
                    Vector2.new(rootPos.X + sizeX/2, rootPos.Y - sizeY),  -- superior direito
                    Vector2.new(rootPos.X + sizeX/2, rootPos.Y),          -- inferior direito
                }

                box.PointA = points[1]
                box.PointB = points[2]
                box.PointC = points[3]
                box.PointD = points[4]

                -- Cor do time
                local teamName = player.Team and player.Team.Name or "Civil"
                local color = teamColors[teamName] or Color3.fromRGB(255, 255, 255)

                box.Color = color
                box.Visible = true
            else
                box.Visible = false
            end
        else
            box.Visible = false
        end
    end
end

local function enableESP()
    espEnabled = true
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Player then
            ESP[player] = createESP(player)
        end
    end
end

local function disableESP()
    espEnabled = false
    for _, box in pairs(ESP) do
        box:Remove()
    end
    ESP = {}
end

Players.PlayerAdded:Connect(function(player)
    if espEnabled and player ~= Player then
        ESP[player] = createESP(player)
    end
end)

Players.PlayerRemoving:Connect(function(player)
    if ESP[player] then
        ESP[player]:Remove()
        ESP[player] = nil
    end
end)

RunService.RenderStepped:Connect(function()
    if espEnabled then
        updateESP()
    end
end)

TabVisual:CreateToggle({
    Name = "ESP por time",
    CurrentValue = false,
    Flag = "ESPToggle",
    Callback = function(value)
        if value then
            local Players = game:GetService("Players")
            local Teams = game:GetService("Teams")

            local localPlayer = Players.LocalPlayer

            local teamColors = {}

            -- Pega as cores dos times
            for _, team in pairs(Teams:GetChildren()) do
                teamColors[team.Name] = team.TeamColor.Color
            end

            -- Cria ou atualiza o Highlight no personagem
            local function setHighlightForCharacter(character, color)
                if not character then return end

                -- Espera o Humanoid para garantir que o personagem está pronto
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if not humanoid then
                    humanoid = character:WaitForChild("Humanoid", 5)
                    if not humanoid then return end
                end

                -- Busca Highlight já existente
                local highlight = character:FindFirstChild("ESP_Highlight")
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "ESP_Highlight"
                    highlight.Parent = character
                end
                highlight.FillTransparency = 1 -- sem preenchimento, só contorno
                highlight.OutlineColor = color
                highlight.Enabled = true
            end

            local function removeHighlight(character)
                if not character then return end
                local highlight = character:FindFirstChild("ESP_Highlight")
                if highlight then
                    highlight.Enabled = false
                    highlight:Destroy()
                end
            end

            local function updateESP(player)
                local character = player.Character
                if character then
                    local teamName = player.Team and player.Team.Name or nil
                    if teamName and teamColors[teamName] then
                        setHighlightForCharacter(character, teamColors[teamName])
                    else
                        removeHighlight(character)
                    end
                end
            end

            -- Quando personagem spawna ou troca de time
            local function onCharacterAdded(player, character)
                updateESP(player)

                -- Atualiza se o time mudar
                player:GetPropertyChangedSignal("Team"):Connect(function()
                    updateESP(player)
                end)
            end

            -- ESP em jogadores já no jogo
            for _, player in pairs(Players:GetPlayers()) do
                if player.Character then
                    onCharacterAdded(player, player.Character)
                end
                player.CharacterAdded:Connect(function(character)
                    onCharacterAdded(player, character)
                end)
            end

            -- ESP em jogadores que entrarem depois
            Players.PlayerAdded:Connect(function(player)
                player.CharacterAdded:Connect(function(character)
                    onCharacterAdded(player, character)
                end)
                player:GetPropertyChangedSignal("Team"):Connect(function()
                    updateESP(player)
                end)
            end)
        else
            local function disableESP()
                for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                    local character = player.Character
                    if character then
                        local highlight = character:FindFirstChild("ESP_Highlight")
                        if highlight then
                            highlight:Destroy() -- Remove o ESP do personagem
                        end
                    end
                end
            end
            disableESP()
        end
    end
})

-- Abre o menu com CTRL
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.LeftControl then
        Window:Toggle()
    end
end)
