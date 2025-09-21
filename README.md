# juamz local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local HttpService = game:GetService("HttpService")

-- CONFIGURAÇÃO: IDs dos donos
local OWNER_IDS = {
    [12345678] = true, -- troque pelo seu UserId
}

-- Funções de segurança e UI (lado do cliente)
local function isOwner(player)
    return OWNER_IDS[player.UserId] == true
end

local function notify(msg)
    game.StarterGui:SetCore("SendNotification", {
        Title = "Admin",
        Text = msg,
        Duration = 3
    })
end

local function findTarget(name)
    for _, player in ipairs(Players:GetPlayers()) do
        if string.sub(string.lower(player.Name), 1, string.len(name)) == string.lower(name) then
            return player
        end
    end
    return nil
end

local function createAdminUI()
    local gui = Instance.new("ScreenGui")
    gui.Name = "AdminMenu"
    gui.ResetOnSpawn = false

    local toggleBtn = Instance.new("TextButton", gui)
    toggleBtn.Size = UDim2.new(0,120,0,40)
    toggleBtn.Position = UDim2.new(0,20,0.8,0)
    toggleBtn.Text = "⚙️ Menu ADM"
    toggleBtn.BackgroundColor3 = Color3.fromRGB(100,0,150)
    toggleBtn.TextColor3 = Color3.new(1,1,1)
    toggleBtn.Font = Enum.Font.SourceSansBold
    toggleBtn.TextSize = 20

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0,220,0,300)
    frame.Position = UDim2.new(0,20,0.5,-150)
    frame.BackgroundColor3 = Color3.fromRGB(60,0,100)
    frame.Visible = false
    frame.Active = true
    frame.Draggable = true

    local uicorner = Instance.new("UICorner", frame)
    uicorner.CornerRadius = UDim.new(0,12)
    local uiStroke = Instance.new("UIStroke", frame)
    uiStroke.Thickness = 2
    uiStroke.Color = Color3.fromRGB(180,0,255)
    local uiList = Instance.new("UIListLayout", frame)
    uiList.Padding = UDim.new(0,5)
    uiList.HorizontalAlignment = Enum.HorizontalAlignment.Center
    uiList.SortOrder = Enum.SortOrder.LayoutOrder

    local targetBox = Instance.new("TextBox", frame)
    targetBox.Size = UDim2.new(0.9,0,0,35)
    targetBox.PlaceholderText = "Nome do player"
    targetBox.BackgroundColor3 = Color3.fromRGB(90,0,140)
    targetBox.TextColor3 = Color3.new(1,1,1)
    targetBox.Font = Enum.Font.SourceSansBold
    targetBox.TextSize = 18
    Instance.new("UICorner", targetBox).CornerRadius = UDim.new(0,8)

    local function createButton(txt, color, action)
        local btn = Instance.new("TextButton", frame)
        btn.Size = UDim2.new(0.9,0,0,40)
        btn.Text = txt
        btn.BackgroundColor3 = color
        btn.TextColor3 = Color3.new(1,1,1)
        btn.Font = Enum.Font.SourceSansBold
        btn.TextSize = 18
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)
        btn.MouseButton1Click:Connect(function()
            if targetBox.Text ~= "" then
                local target = findTarget(targetBox.Text)
                if target and target.Character and not isOwner(target) then
                    action(target)
                    notify(txt.." executado em "..target.Name)
                else
                    notify("Ação inválida.")
                end
            end
        end)
    end

    -- Handlers das ações (lado do cliente)
    local function jail(plr)
        local rootPart = plr.Character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            rootPart.CFrame = CFrame.new(0,100,0) -- Teleporta para um lugar alto
            plr.Character.Humanoid.WalkSpeed = 0
            plr.Character.Humanoid.JumpPower = 0
        end
    end

    local function unjail(plr)
        plr.Character.Humanoid.WalkSpeed = 16
        plr.Character.Humanoid.JumpPower = 50
    end

    local function freeze(plr)
        local rootPart = plr.Character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            rootPart.Anchored = true
        end
    end

    local function unfreeze(plr)
        local rootPart = plr.Character:FindFirstChild("HumanoidRootPart")
        if rootPart then
            rootPart.Anchored = false
        end
    end

    local function kick(plr)
        -- Ação de kick não funciona em executores de forma segura.
        -- É comum em jogos de roblox fazer algo como:
        -- game.ReplicatedStorage.RemoteEvents.Kick:FireServer(plr)
        -- mas isso só funciona se o jogo já tiver um evento remoto para kick
        notify("Kick não é possível em um executor de forma confiável.")
    end

    createButton("Jail", Color3.fromRGB(120,0,180), jail)
    createButton("Unjail", Color3.fromRGB(120,0,180), unjail)
    createButton("Freeze", Color3.fromRGB(120,0,180), freeze)
    createButton("Unfreeze", Color3.fromRGB(120,0,180), unfreeze)
    createButton("Kick", Color3.fromRGB(200,0,100), kick)

    toggleBtn.MouseButton1Click:Connect(function()
        frame.Visible = not frame.Visible
    end)

    gui.Parent = game.CoreGui
end

-- Roda o script se o jogador atual for um dono
if isOwner(Players.LocalPlayer) then
    createAdminUI()
else
    notify("Você não é o dono e não pode usar o menu de admin.")
end
