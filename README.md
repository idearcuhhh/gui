local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

-- Mapping transformations to their display names
local transformationMap = {
    ["Divine Rose Prominence"] = "[DRP]",
    ["Astral Instinct"] = "[AI]",
    ["Ultra Instinct"] = "[UI]",
    ["Mastered Ultra Instinct"] = "[MUI]",
    ["Ultra Ego"] = "[UE]",
    ["God of Destruction"] = "[GOD]",
    ["God of Creation"] = "[GOC]",
    ["Divine Blue"] = "[DB]",
    ["Jiren Ultra Instinct"] = "[JUI]"
}

-- Function to abbreviate numbers (e.g., 1000000 -> 1M)
local function abbreviateNumber(num)
    local suffixes = { "", "K", "M", "B", "T", "Qa", "Qi", "Sx", "Sp", "Oc", "No", "Dc" }
    local index = 1
    while num >= 1000 and index < #suffixes do
        num = num / 1000
        index = index + 1
    end
    return string.format("%.1f%s", num, suffixes[index]):gsub("%.0", "") -- Remove .0 for whole numbers
end

-- Function to determine the color based on health value
local function getHealthColor(health)
    if health < 1000 then
        return Color3.fromRGB(0, 255, 0) -- Green
    elseif health < 1_000_000 then
        return Color3.fromRGB(255, 255, 0) -- Yellow
    elseif health < 1_000_000_000 then
        return Color3.fromRGB(255, 0, 0) -- Red
    elseif health < 1_000_000_000_000 then
        return Color3.fromRGB(0, 0, 255) -- Blue
    elseif health < 1_000_000_000_000_000 then
        return Color3.fromRGB(0, 255, 255) -- Light Blue
    else
        return Color3.fromRGB(128, 0, 128) -- Purple
    end
end

local function getPlayerTransformation(player)
    local playerFolder = Workspace:FindFirstChild("Players")
    if playerFolder then
        local playerStatus = playerFolder:FindFirstChild(player.Name)
        if playerStatus then
            local status = playerStatus:FindFirstChild("Status") -- Fixed capitalization
            if status then
                local transformation = status:FindFirstChild("Transformation")
                if transformation and transformation.Value ~= "" then
                    return transformationMap[transformation.Value] or "[Base]" -- Default to [Base] if not mapped
                end
            end
        end
    end
    return "[Base]"
end

local function createBillboard(player)
    if not player.Character or not player.Character:FindFirstChild("Head") then return end

    local character = player.Character
    local head = character:FindFirstChild("Head")

    -- Remove existing BillboardGuis to prevent duplicates
    if head:FindFirstChild("NameTag") then
        head:FindFirstChild("NameTag"):Destroy()
    end

    -- Create BillboardGui
    local billboard = Instance.new("BillboardGui")
    billboard.Name = "NameTag"
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
    billboard.AlwaysOnTop = true
    billboard.Adornee = head
    billboard.Parent = head

    -- Create TextLabel
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.TextStrokeTransparency = 0
    textLabel.Font = Enum.Font.SourceSansBold
    textLabel.TextSize = 20
    textLabel.TextScaled = false
    textLabel.Parent = billboard

    -- Function to update text and color continuously
    task.spawn(function()
        while player and player.Character and player.Character:FindFirstChild("Humanoid") do
            local humanoid = player.Character:FindFirstChild("Humanoid")
            local currentHealth = math.floor(humanoid.Health)
            local maxHealth = math.floor(humanoid.MaxHealth)

            local currentHealthText = abbreviateNumber(currentHealth)
            local maxHealthText = abbreviateNumber(maxHealth)

            local transformation = getPlayerTransformation(player)

            textLabel.Text = string.format("%s %s [@%s] | [%s/%s HP]", transformation, player.DisplayName, player.Name, currentHealthText, maxHealthText)
            textLabel.TextColor3 = getHealthColor(currentHealth) -- Change text color dynamically
            
            task.wait(0) -- Allows smooth updates without overwhelming performance
        end
    end)
end

local function onCharacterAdded(character)
    local player = Players:GetPlayerFromCharacter(character)
    if player then
        task.wait(1) -- Small delay to ensure character is loaded
        createBillboard(player)
    end
end

local function onPlayerAdded(player)
    if player.Character then
        createBillboard(player)
    end
    player.CharacterAdded:Connect(onCharacterAdded)
end

-- Connect to all players
for _, player in ipairs(Players:GetPlayers()) do
    onPlayerAdded(player)
end

Players.PlayerAdded:Connect(onPlayerAdded)
