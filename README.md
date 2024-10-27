-- Interface GUI Completa para Farm até Nível Máximo

local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local TitleLabel = Instance.new("TextLabel")
local ToggleFarmButton = Instance.new("TextButton")
local StatusLabel = Instance.new("TextLabel")
local LevelLabel = Instance.new("TextLabel")
local RefreshButton = Instance.new("TextButton")

ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.Position = UDim2.new(0.1, 0, 0.7, 0)
MainFrame.Size = UDim2.new(0, 250, 0, 150)

TitleLabel.Parent = MainFrame
TitleLabel.BackgroundTransparency = 1
TitleLabel.Size = UDim2.new(0, 250, 0, 30)
TitleLabel.Position = UDim2.new(0, 0, 0, 10)
TitleLabel.Text = "Painel de Controle de Farm"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.TextSize = 18
TitleLabel.Font = Enum.Font.SourceSansBold

ToggleFarmButton.Parent = MainFrame
ToggleFarmButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
ToggleFarmButton.Size = UDim2.new(0, 230, 0, 30)
ToggleFarmButton.Position = UDim2.new(0, 10, 0, 50)
ToggleFarmButton.Text = "Iniciar Farm"
ToggleFarmButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleFarmButton.TextSize = 16
ToggleFarmButton.Font = Enum.Font.SourceSansBold

StatusLabel.Parent = MainFrame
StatusLabel.BackgroundTransparency = 1
StatusLabel.Size = UDim2.new(0, 230, 0, 20)
StatusLabel.Position = UDim2.new(0, 10, 0, 90)
StatusLabel.Text = "Status: Inativo"
StatusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
StatusLabel.TextSize = 14

LevelLabel.Parent = MainFrame
LevelLabel.BackgroundTransparency = 1
LevelLabel.Size = UDim2.new(0, 230, 0, 20)
LevelLabel.Position = UDim2.new(0, 10, 0, 110)
LevelLabel.Text = "Nível Atual: "
LevelLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
LevelLabel.TextSize = 14

RefreshButton.Parent = MainFrame
RefreshButton.BackgroundColor3 = Color3.fromRGB(70, 130, 180)
RefreshButton.Size = UDim2.new(0, 230, 0, 20)
RefreshButton.Position = UDim2.new(0, 10, 0, 130)
RefreshButton.Text = "Atualizar Nível"
RefreshButton.TextColor3 = Color3.fromRGB(255, 255, 255)
RefreshButton.TextSize = 14
RefreshButton.Font = Enum.Font.SourceSans

local farming = false

local questData = {
    {level = 0, questName = "Bandit Quest", enemyName = "Bandit", npcName = "BanditQuestGiverNPC"},
    {level = 10, questName = "Monkey Quest", enemyName = "Monkey", npcName = "MonkeyQuestGiverNPC"},
    {level = 30, questName = "Gorilla Quest", enemyName = "Gorilla", npcName = "GorillaQuestGiverNPC"},
    {level = 50, questName = "Pirate Quest", enemyName = "Pirate", npcName = "PirateQuestGiverNPC"},
}

local attackRange = 50
local refreshDelay = 2

function moveTo(position)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    character.Humanoid:MoveTo(position)
end

function startQuest(npcName)
    local npc = game.Workspace:FindFirstChild(npcName)
    if npc then
        moveTo(npc.Position)
        wait(1)
        fireproximityprompt(npc.ProximityPrompt)
    end
end

function attackEnemy(enemy)
    if enemy and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()
        
        character.HumanoidRootPart.CFrame = enemy.HumanoidRootPart.CFrame * CFrame.new(0, 0, -5)
        wait(0.5)
        
        character:FindFirstChildOfClass("Tool"):Activate()
    end
end

function farmToMaxLevel()
    while farming do
        local player = game.Players.LocalPlayer
        local playerLevel = player.Data.Level.Value
        local quest = nil

        for i = #questData, 1, -1 do
            if playerLevel >= questData[i].level then
                quest = questData[i]
                break
            end
        end

        local questActive = player.PlayerGui.QuestTracker:FindFirstChild(quest.questName)
        if not questActive then
            startQuest(quest.npcName)
        end

        for _, enemy in pairs(game.Workspace.Enemies:GetChildren()) do
            if enemy.Name == quest.enemyName and (enemy.HumanoidRootPart.Position - player.Character.HumanoidRootPart.Position).magnitude <= attackRange then
                attackEnemy(enemy)
                wait(1)
            end
        end

        LevelLabel.Text = "Nível Atual: " .. playerLevel
        wait(refreshDelay)
    end
end

ToggleFarmButton.MouseButton1Click:Connect(function()
    farming = not farming
    if farming then
        StatusLabel.Text = "Status: Ativo"
        ToggleFarmButton.Text = "Parar Farm"
        farmToMaxLevel()
    else
        StatusLabel.Text = "Status: Inativo"
        ToggleFarmButton.Text = "Iniciar Farm"
    end
end)

RefreshButton.MouseButton1Click:Connect(function()
    local playerLevel = game.Players.LocalPlayer.Data.Level.Value
    LevelLabel.Text = "Nível Atual: " .. playerLevel
end)
