-- Script Auto Fly + Auto Attack (FIXED NO CLIP)
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local plr = Players.LocalPlayer
local flyActive = true
local targetPlayer = nil
local flyConnection = nil
local block = nil
local heightOffset = 5
local blacklist = {}
local healthCheckConnection = nil
local hopPending = false
local isRespawning = false

-- Biến cho auto attack và auto skills
_G.AutoAttack = true
_G.AttackPlayers = true
_G.RaceClickAutov4 = true
_G.SelectWeapon = nil
local Sec = 1 -- Delay cho Buso
local Boud = true -- Auto Buso

-- Tạo UI Bên Trái
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyControlUI"
screenGui.Parent = CoreGui
screenGui.ResetOnSpawn = false

-- Container chính (bên trái màn hình)
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 280, 0, 220)
mainFrame.Position = UDim2.new(0, 10, 0.5, -110)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BackgroundTransparency = 0.2
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Name = "Title"
title.Size = UDim2.new(1, 0, 0, 35)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
title.BackgroundTransparency = 0.3
title.Text = "🚀 AUTO FLY"
title.TextColor3 = Color3.fromRGB(0, 191, 255)
title.TextSize = 16
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = title

local playerFrame = Instance.new("Frame")
playerFrame.Name = "PlayerFrame"
playerFrame.Size = UDim2.new(1, -20, 0, 50)
playerFrame.Position = UDim2.new(0, 10, 0, 45)
playerFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
playerFrame.BackgroundTransparency = 0.3
playerFrame.Parent = mainFrame

local playerCorner = Instance.new("UICorner")
playerCorner.CornerRadius = UDim.new(0, 6)
playerCorner.Parent = playerFrame

local targetIcon = Instance.new("ImageLabel")
targetIcon.Name = "TargetIcon"
targetIcon.Size = UDim2.new(0, 24, 0, 24)
targetIcon.Position = UDim2.new(0, 10, 0.5, -12)
targetIcon.BackgroundTransparency = 1
targetIcon.Image = "rbxassetid://3926305904"
targetIcon.ImageRectOffset = Vector2.new(964, 324)
targetIcon.ImageRectSize = Vector2.new(36, 36)
targetIcon.ImageColor3 = Color3.fromRGB(0, 191, 255)
targetIcon.Parent = playerFrame

local playerNameLabel = Instance.new("TextLabel")
playerNameLabel.Name = "PlayerName"
playerNameLabel.Size = UDim2.new(1, -50, 1, 0)
playerNameLabel.Position = UDim2.new(0, 40, 0, 0)
playerNameLabel.BackgroundTransparency = 1
playerNameLabel.Text = "Đang tìm target..."
playerNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
playerNameLabel.TextSize = 14
playerNameLabel.Font = Enum.Font.Gotham
playerNameLabel.TextXAlignment = Enum.TextXAlignment.Left
playerNameLabel.Parent = playerFrame

local healthLabel = Instance.new("TextLabel")
healthLabel.Name = "HealthLabel"
healthLabel.Size = UDim2.new(1, -50, 0, 20)
healthLabel.Position = UDim2.new(0, 40, 0, 25)
healthLabel.BackgroundTransparency = 1
healthLabel.Text = "Máu: --"
healthLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
healthLabel.TextSize = 12
healthLabel.Font = Enum.Font.GothamMedium
healthLabel.TextXAlignment = Enum.TextXAlignment.Left
healthLabel.Parent = playerFrame

local buttonContainer = Instance.new("Frame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(1, -20, 0, 110)
buttonContainer.Position = UDim2.new(0, 10, 0, 105)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Parent = mainFrame

local function createButton(name, text, color, position)
    local btn = Instance.new("TextButton")
    btn.Name = name
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.Position = position
    btn.BackgroundColor3 = color
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 14
    btn.Font = Enum.Font.GothamBold
    btn.Parent = buttonContainer
    
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 6)
    btnCorner.Parent = btn
    
    return btn
end

local blacklistBtn = createButton("BlacklistBtn", "➕ Thêm Blacklist", Color3.fromRGB(255, 100, 100), UDim2.new(0, 0, 0, 0))
local hopBtn = createButton("HopBtn", "🔄 Hop Server", Color3.fromRGB(100, 100, 255), UDim2.new(0, 0, 0, 35))
local infoLabel = createButton("InfoLabel", "🟢 Auto: ON | ⚔️ Attack: ON", Color3.fromRGB(60, 60, 70), UDim2.new(0, 0, 0, 70))
infoLabel.Text = "🟢 Auto: ON | ⚔️ Attack: ON"
infoLabel.AutoButtonColor = false

-- ==================== FIXED NO CLIP SYSTEM ====================

local noclipConnection = nil
local originalCanCollide = {}

-- Hàm bật noclip mạnh mẽ và ổn định
function enableNoclip()
    if not plr.Character then return end
    
    -- Hủy noclip cũ nếu có
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    
    -- Xóa BodyVelocity cũ
    local bodyVelocity = plr.Character:FindFirstChild("HumanoidRootPart"):FindFirstChild("BodyClip")
    if bodyVelocity then
        bodyVelocity:Destroy()
    end
    
    -- Lưu trạng thái CanCollide gốc và đặt thành false
    for _, part in pairs(plr.Character:GetDescendants()) do
        if part:IsA("BasePart") then
            originalCanCollide[part] = part.CanCollide
            part.CanCollide = false
        end
    end
    
    -- Tạo BodyVelocity mới
    local noclip = Instance.new("BodyVelocity")
    noclip.Name = "BodyClip"
    noclip.Parent = plr.Character.HumanoidRootPart
    noclip.MaxForce = Vector3.new(100000, 100000, 100000)
    noclip.Velocity = Vector3.new(0, 0, 0)
    
    -- Kết nối Heartbeat để duy trì noclip
    noclipConnection = RunService.Heartbeat:Connect(function()
        if not plr.Character or not flyActive then 
            if noclipConnection then
                noclipConnection:Disconnect()
            end
            return 
        end
        
        -- Đảm bảo tất cả parts không thể va chạm
        for _, part in pairs(plr.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        
        -- Đảm bảo BodyVelocity tồn tại
        if not plr.Character.HumanoidRootPart:FindFirstChild("BodyClip") then
            local newNoclip = Instance.new("BodyVelocity")
            newNoclip.Name = "BodyClip"
            newNoclip.Parent = plr.Character.HumanoidRootPart
            newNoclip.MaxForce = Vector3.new(100000, 100000, 100000)
            newNoclip.Velocity = Vector3.new(0, 0, 0)
        end
    end)
    
    print("✅ Noclip đã được bật")
end

-- Hàm tắt noclip
function disableNoclip()
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end
    
    if not plr.Character then return end
    
    -- Xóa BodyVelocity
    local bodyVelocity = plr.Character:FindFirstChild("HumanoidRootPart"):FindFirstChild("BodyClip")
    if bodyVelocity then
        bodyVelocity:Destroy()
    end
    
    -- Khôi phục CanCollide
    for part, canCollide in pairs(originalCanCollide) do
        if part and part.Parent then
            part.CanCollide = canCollide
        end
    end
    originalCanCollide = {}
    
    print("❌ Noclip đã tắt")
end

-- ==================== AUTO ATTACK FIXED FUNCTIONS ====================

-- Hàm lấy tất cả mục tiêu (bao gồm cả quái và người chơi)
local function getAllTargets(position, radius)
    local targets = {}
    
    -- Tìm quái vật trong Workspace
    local function findEnemiesInFolder(folderName)
        local folder = Workspace:FindFirstChild(folderName)
        if folder then
            for _, enemy in ipairs(folder:GetChildren()) do
                if enemy:IsA("Model") then
                    local humanoid = enemy:FindFirstChild("Humanoid")
                    local rootPart = enemy:FindFirstChild("HumanoidRootPart")
                    if humanoid and rootPart and humanoid.Health > 0 then
                        local distance = (rootPart.Position - position).Magnitude
                        if distance <= radius then
                            table.insert(targets, {Model = enemy, Type = "Mob"})
                        end
                    end
                end
            end
        end
    end
    
    -- Tìm trong các folder có thể chứa quái
    findEnemiesInFolder("Enemies")
    findEnemiesInFolder("_ENEMIES")
    findEnemiesInFolder("Mobs")
    findEnemiesInFolder("NPCs")
    
    -- Tìm trực tiếp trong Workspace
    for _, enemy in ipairs(Workspace:GetChildren()) do
        if enemy:IsA("Model") and enemy.Name:lower():find("bandit") 
           or enemy.Name:lower():find("pirate") 
           or enemy.Name:lower():find("assassin") 
           or enemy.Name:lower():find("marine") then
            
            local humanoid = enemy:FindFirstChild("Humanoid")
            local rootPart = enemy:FindFirstChild("HumanoidRootPart")
            if humanoid and rootPart and humanoid.Health > 0 then
                local distance = (rootPart.Position - position).Magnitude
                if distance <= radius then
                    table.insert(targets, {Model = enemy, Type = "Mob"})
                end
            end
        end
    end
    
    -- Lấy người chơi khác nếu được bật
    if _G.AttackPlayers then
        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= plr and otherPlayer.Character then
                local character = otherPlayer.Character
                local humanoid = character:FindFirstChild("Humanoid")
                local rootPart = character:FindFirstChild("HumanoidRootPart")
                if humanoid and rootPart and humanoid.Health > 0 then
                    local distance = (rootPart.Position - position).Magnitude
                    if distance <= radius then
                        table.insert(targets, {Model = character, Type = "Player"})
                    end
                end
            end
        end
    end
    
    return targets
end

-- Hàm tấn công đơn giản (dùng RemoteEvent)
local function simpleAttack(target)
    if not target or not target.Model then return end
    
    local character = plr.Character
    if not character then return end
    
    -- Tìm weapon
    local weapon = character:FindFirstChildOfClass("Tool")
    if not weapon then
        -- Kiểm tra trong backpack
        if plr.Backpack then
            for _, tool in ipairs(plr.Backpack:GetChildren()) do
                if tool:IsA("Tool") then
                    weapon = tool
                    tool.Parent = character
                    wait(0.1)
                    break
                end
            end
        end
    end
    
    if not weapon then return end
    
    -- Tìm remote để tấn công
    local function findAttackRemote()
        -- Thử tìm trong ReplicatedStorage
        local modules = ReplicatedStorage:FindFirstChild("Modules")
        if modules then
            local net = modules:FindFirstChild("Net")
            if net then
                local registerAttack = net:FindFirstChild("RE/RegisterAttack")
                local registerHit = net:FindFirstChild("RE/RegisterHit")
                if registerAttack and registerHit then
                    return registerAttack, registerHit
                end
            end
        end
        
        -- Thử tìm trực tiếp trong ReplicatedStorage
        local remotes = ReplicatedStorage:GetChildren()
        for _, remote in ipairs(remotes) do
            if remote:IsA("RemoteEvent") then
                if remote.Name:lower():find("attack") 
                   or remote.Name:lower():find("hit") 
                   or remote.Name:lower():find("damage") 
                   or remote.Name:lower():find("combat") then
                    return remote
                end
            end
        end
        
        return nil
    end
    
    -- Thực hiện tấn công
    local attackRemote, hitRemote = findAttackRemote()
    if attackRemote and hitRemote then
        -- Fire registerAttack
        pcall(function()
            attackRemote:FireServer()
        end)
        
        -- Chọn bộ phận để tấn công
        local hitPart = target.Model:FindFirstChild("HumanoidRootPart") 
                       or target.Model:FindFirstChild("Head") 
                       or target.Model:FindFirstChild("Torso")
                       or target.Model.PrimaryPart
        
        if hitPart then
            pcall(function()
                -- Tạo hitTargets array
                local hitTargets = {{target.Model, hitPart}}
                hitRemote:FireServer(hitPart, hitTargets)
                
                -- Gửi click event để kích hoạt attack animation
                if weapon:FindFirstChild("Activated") then
                    weapon.Activated:Fire()
                end
                
                -- Fire click remote nếu có
                local clickRemote = weapon:FindFirstChild("ClickRemote") 
                                  or weapon:FindFirstChild("LeftClickRemote")
                                  or weapon:FindFirstChild("RightClickRemote")
                
                if clickRemote then
                    clickRemote:FireServer()
                end
            end)
        end
    elseif attackRemote then
        -- Nếu chỉ có 1 remote
        pcall(function()
            attackRemote:FireServer(target.Model)
        end)
    end
    
    -- Thử kích hoạt tool trực tiếp
    if weapon:FindFirstChild("Activated") then
        pcall(function()
            weapon.Activated:Fire()
        end)
    end
end

-- Hàm kiểm tra và tấn công mục tiêu
local function checkAndAttack()
    if not _G.AutoAttack then return end
    
    local character = plr.Character
    if not character then return end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return end
    
    -- Lấy tất cả mục tiêu trong bán kính 50 studs
    local targets = getAllTargets(rootPart.Position, 50)
    
    if #targets > 0 then
        -- Chọn mục tiêu gần nhất
        local nearestTarget = nil
        local nearestDistance = math.huge
        
        for _, target in ipairs(targets) do
            local targetRoot = target.Model:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local distance = (targetRoot.Position - rootPart.Position).Magnitude
                if distance < nearestDistance then
                    nearestDistance = distance
                    nearestTarget = target
                end
            end
        end
        
        if nearestTarget then
            -- Đảm bảo đang nhìn về mục tiêu
            local targetRoot = nearestTarget.Model:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                character:SetPrimaryPartCFrame(
                    CFrame.new(rootPart.Position, Vector3.new(
                        targetRoot.Position.X, 
                        rootPart.Position.Y, 
                        targetRoot.Position.Z
                    ))
                )
                
                -- Thực hiện tấn công
                simpleAttack(nearestTarget)
                
                -- Log để debug
                print("⚔️ Đang tấn công: " .. nearestTarget.Model.Name .. " (" .. nearestTarget.Type .. ")")
            end
        end
    end
end

-- ==================== FLY FUNCTIONS ====================

-- Tạo block ảo cho tween
function createVirtualBlock()
    if block and block.Parent then 
        block:Destroy() 
    end
    
    block = Instance.new("Part")
    block.Name = "VirtualFlyBlock"
    block.Size = Vector3.new(1, 1, 1)
    block.Anchored = true
    block.CanCollide = false
    block.Transparency = 1
    block.Parent = workspace
    
    return block
end

-- Tìm người chơi gần nhất không trong danh sách đen
function getNearestPlayer()
    local nearestPlayer = nil
    local nearestDistance = math.huge
    local localRoot = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
    
    if not localRoot then return nil end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= plr and not blacklist[player] then
            if player.Character then
                local targetRoot = player.Character:FindFirstChild("HumanoidRootPart")
                if targetRoot then
                    local humanoid = player.Character:FindFirstChild("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        local distance = (targetRoot.Position - localRoot.Position).Magnitude
                        if distance < nearestDistance then
                            nearestDistance = distance
                            nearestPlayer = player
                        end
                    else
                        blacklist[player] = true
                    end
                end
            end
        end
    end
    
    return nearestPlayer
end

-- Kiểm tra máu của target player
function checkTargetHealth()
    if not targetPlayer or not targetPlayer.Character then
        return false
    end
    
    local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
    if not humanoid or humanoid.Health <= 0 then
        blacklist[targetPlayer] = true
        return false
    end
    
    return true
end

-- Bật highlight
function enableHighlight()
    if not plr.Character then return end
    
    if plr.Character:FindFirstChild("highlight") then
        plr.Character:FindFirstChild("highlight"):Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "highlight"
    highlight.Enabled = true
    highlight.FillColor = Color3.fromRGB(0, 191, 255)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0.2
    highlight.Parent = plr.Character
end

-- Tính toán vị trí đứng trên đầu player
function getHeadPosition(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then
        return nil
    end
    
    local character = targetPlayer.Character
    local head = character:FindFirstChild("Head")
    if head then
        return head.Position + Vector3.new(0, heightOffset, 0)
    end
    
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if rootPart then
        return rootPart.Position + Vector3.new(0, 4 + heightOffset, 0)
    end
    
    return nil
end

-- Hàm bay không delay - bám sát target và đứng trên đầu
function startFlying()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    if not block then
        block = createVirtualBlock()
    end
    
    local character = plr.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        return
    end
    
    local rootPart = character.HumanoidRootPart
    
    -- Đảm bảo noclip đang hoạt động trước khi bay
    if not noclipConnection then
        enableNoclip()
    end
    
    -- Cập nhật vị trí block theo real-time
    flyConnection = RunService.Heartbeat:Connect(function(deltaTime)
        if not flyActive or not targetPlayer or not targetPlayer.Character then
            return
        end
        
        if not checkTargetHealth() then
            local newTarget = getNearestPlayer()
            if newTarget then
                targetPlayer = newTarget
                updateUI()
            else
                return
            end
        end
        
        local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        if not targetRoot then
            return
        end
        
        -- Đảm bảo noclip luôn hoạt động trong khi bay
        for _, part in pairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
        
        local targetHeadPos = getHeadPosition(targetPlayer)
        if not targetHeadPos then
            targetHeadPos = targetRoot.Position + Vector3.new(0, heightOffset, 0)
        end
        
        local currentPos = rootPart.Position
        local direction = (targetHeadPos - currentPos).Unit
        local distance = (targetHeadPos - currentPos).Magnitude
        
        local speed = 300
        local moveDistance = math.min(speed * deltaTime, distance)
        
        if distance > 2 then
            local newPos = currentPos + (direction * moveDistance)
            local lookAtPos = Vector3.new(targetRoot.Position.X, newPos.Y, targetRoot.Position.Z)
            rootPart.CFrame = CFrame.new(newPos, lookAtPos)
            block.CFrame = rootPart.CFrame
        else
            local finalLookAt = Vector3.new(targetRoot.Position.X, targetHeadPos.Y, targetRoot.Position.Z)
            rootPart.CFrame = CFrame.new(targetHeadPos, finalLookAt)
            block.CFrame = rootPart.CFrame
        end
    end)
end

-- Dừng bay
function stopFlying()
    flyActive = false
    
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    
    if noclipConnection then
        disableNoclip()
    end
    
    print("✈️ Đã dừng bay")
end

-- Cập nhật UI
function updateUI()
    if targetPlayer then
        playerNameLabel.Text = targetPlayer.Name
        if targetPlayer.Character then
            local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
            if humanoid then
                healthLabel.Text = "Máu: " .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth)
            end
        end
    else
        playerNameLabel.Text = "Đang tìm target..."
        healthLabel.Text = "Máu: --"
    end
end

-- AUTO SKILLS FUNCTIONS ------------------------------------------------------

-- Tự động trang bị Blox Fruit
local function equipBloxFruit()
    if plr.Backpack then
        for _, v in pairs(plr.Backpack:GetChildren()) do
            if v.ToolTip == "Blox Fruit" then
                if plr.Backpack:FindFirstChild(tostring(v.Name)) then
                    _G.SelectWeapon = v.Name
                    v.Parent = plr.Character
                    print("🍇 Đã trang bị Blox Fruit: " .. v.Name)
                    break
                end
            end
        end
    end
end

-- Auto Buso Haki
local function autoBuso()
    pcall(function()
        if Boud then
            local _HasBuso = {"HasBuso", "Buso"}
            if not plr.Character:FindFirstChild(_HasBuso[1]) then
                ReplicatedStorage.Remotes.CommF_:InvokeServer(_HasBuso[2])
                print("🛡️ Đã bật Buso Haki")
            end
        end
    end)
end

-- Auto Race V4
local function autoRaceV4()
    pcall(function()
        if _G.RaceClickAutov4 then
            if plr.Character:FindFirstChild("RaceEnergy") then
                if plr.Character:FindFirstChild("RaceEnergy").Value == 1 then
                    print("⚡ Đã kích hoạt Race V4")
                end
            end
        end
    end)
end

-- BUTTON FUNCTIONS -----------------------------------------------------------

blacklistBtn.MouseButton1Click:Connect(function()
    if targetPlayer then
        blacklist[targetPlayer] = true
        print("🚫 Đã thêm " .. targetPlayer.Name .. " vào blacklist")
        
        local newTarget = getNearestPlayer()
        if newTarget then
            targetPlayer = newTarget
            updateUI()
        else
            targetPlayer = nil
            updateUI()
        end
    end
end)

hopBtn.MouseButton1Click:Connect(function()
    print("🔄 Đang tìm server mới...")
    -- Thêm logic hop server ở đây
end)

-- HỆ THỐNG HỒI SINH ---------------------------------------------------------

local function onCharacterAdded(character)
    isRespawning = true
    print("🔄 Đang hồi sinh...")
    
    -- Chờ character load hoàn tất
    wait(2)
    
    if flyActive then
        print("🚀 Tiếp tục bay sau khi hồi sinh...")
        
        -- Bật lại các hiệu ứng
        enableNoclip()
        enableHighlight()
        
        -- Tìm target mới
        local newTarget = getNearestPlayer()
        if newTarget then
            targetPlayer = newTarget
            updateUI()
        end
        
        -- Bắt đầu bay lại
        startFlying()
        
        -- Trang bị tool và bật skills
        equipBloxFruit()
        
        isRespawning = false
        print("✅ Hồi sinh hoàn tất, tiếp tục bay!")
    end
end

local function onCharacterRemoving()
    print("🗑️ Character đang bị xóa...")
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    if noclipConnection then
        disableNoclip()
    end
end

-- KHỞI ĐỘNG SCRIPT ----------------------------------------------------------

local function startAutoFly()
    print("========================================")
    print("🚀 AUTO FLY + ATTACK SCRIPT ĐANG KHỞI ĐỘNG")
    print("========================================")
    
    if not plr.Character then
        print("⏳ Đang chờ character...")
        plr.CharacterAdded:Wait()
    end
    wait(2)
    
    -- Bật các hiệu ứng
    enableNoclip()
    enableHighlight()
    
    -- Trang bị Blox Fruit ngay khi vào game
    equipBloxFruit()
    
    -- Tìm target đầu tiên
    targetPlayer = getNearestPlayer()
    
    if targetPlayer then
        print("🎯 Đã tìm thấy target: " .. targetPlayer.Name)
        updateUI()
        
        -- Bắt đầu bay
        startFlying()
        
        -- Theo dõi thay đổi target
        task.spawn(function()
            while flyActive do
                if not targetPlayer or blacklist[targetPlayer] then
                    local newTarget = getNearestPlayer()
                    if newTarget then
                        targetPlayer = newTarget
                        updateUI()
                        print("🔄 Đã chuyển sang target mới: " .. targetPlayer.Name)
                    end
                end
                wait(0.5)
            end
        end)
    end
    
    -- AUTO SKILLS LOOP
    task.spawn(function()
        while true do
            if plr.Character then
                autoBuso()
                autoRaceV4()
            end
            wait(Sec)
        end
    end)
    
    -- AUTO ATTACK LOOP (FIXED) - Sử dụng Heartbeat để attack liên tục
    task.spawn(function()
        while true do
            if _G.AutoAttack and plr.Character then
                pcall(function()
                    checkAndAttack()
                end)
            end
            wait(0.1) -- Giảm delay để attack nhanh hơn
        end
    end)
    
    print("✅ Script đã khởi động thành công!")
    print("⚔️ Auto Attack: " .. (_G.AutoAttack and "BẬT" or "TẮT"))
    print("🎯 Attack Players: " .. (_G.AttackPlayers and "BẬT" or "TẮT"))
    print("🛡️ Noclip: BẬT (ổn định)")
end

-- KẾT NỐI SỰ KIỆN -----------------------------------------------------------

-- Kết nối sự kiện hồi sinh
plr.CharacterAdded:Connect(onCharacterAdded)
plr.CharacterRemoving:Connect(onCharacterRemoving)

-- Kết nối sự kiện Input để toggle noclip (phím N)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.N then
        if noclipConnection then
            disableNoclip()
        else
            enableNoclip()
        end
    end
end)

-- KHỞI CHẠY CHÍNH -----------------------------------------------------------

task.spawn(function()
    startAutoFly()
end)

print("========================================")
print("🎮 AUTO FLY + ATTACK SCRIPT ĐÃ SẴN SÀNG")
print("========================================")
print("🌟 Tính năng chính:")
print("1. Tự động bay đến player gần nhất")
print("2. Auto Attack quái vật & player")
print("3. Auto trang bị Blox Fruit")
print("4. Auto Buso Haki")
print("5. Auto Race V4")
print("6. Tự động hồi sinh và tiếp tục")
print("7. Noclip ổn định (nhấn N để bật/tắt)")
print("========================================")    {Name = "Quests", Title = "Items Farm", Icon = "📦"},
    {Name = "New", Title = "New Events", Icon = "✨"},
    {Name = "SeaEvent", Title = "Sea Events", Icon = "🌊"},
    {Name = "Mirage", Title = "Mirage + RaceV4", Icon = "🌀"},
    {Name = "Drago", Title = "Drago Dojo", Icon = "🐉"},
    {Name = "Prehistoric", Title = "Prehistoric", Icon = "🦖"},
    {Name = "Raids", Title = "Raid", Icon = "⚔️"},
    {Name = "Combat", Title = "Combat PVP", Icon = "🎯"},
    {Name = "Travel", Title = "Teleport", Icon = "📍"},
    {Name = "Fruit", Title = "Fruits", Icon = "🍎"},
    {Name = "Shop", Title = "Shop", Icon = "🛒"},
    {Name = "Misc", Title = "Misc", Icon = "🔧"}
}

-- ================================================
-- HỆ THỐNG AUTO BONES (WORLD 3)
-- ================================================

-- PHÁT HIỆN THẾ GIỚI
local worldName = ""
local World1, World2, World3 = false, false, false

local function GetWorldName()
    local success, productInfo = pcall(function()
        return MarketplaceService:GetProductInfo(game.PlaceId)
    end)
    if success and productInfo then
        return productInfo.Name
    end
    return "Unknown World"
end

local function DetectWorld()
    worldName = GetWorldName()
    print("World detected:", worldName)
    
    if string.find(worldName, "Blox Fruits$") or worldName == "Blox Fruits" then
        World1 = true
        World2 = false
        World3 = false
        print("World 1 detected")
    elseif string.find(worldName, "Second Sea") then
        World1 = false
        World2 = true
        World3 = false
        print("World 2 detected")
    elseif string.find(worldName, "Third Sea") then
        World1 = false
        World2 = false
        World3 = true
        print("World 3 detected")
    else
        print("Unknown world. Defaulting to World 1")
        World1 = true
        World2 = false
        World3 = false
    end
end

DetectWorld()

-- BIẾN CẤU HÌNH
local ChooseWP = "Melee"
local Boud = true
local BringMobRadius = 300

-- AUTO JOIN MARINES
local HasJoinedMarines = false
local function JoinMarines()
    if HasJoinedMarines then return end
    pcall(function()
        if ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_") then
            ReplicatedStorage.Remotes.CommF_:InvokeServer("SetTeam", "Marines")
        end
    end)
    HasJoinedMarines = true
end

-- CHỌN VÀ EQUIP VŨ KHÍ
local function EquipWeapon()
    if ChooseWP == "Melee" then
        local char = Player.Character
        if not char then return end
        local humanoid = char:FindFirstChild("Humanoid")
        if not humanoid then return end
        
        for _, v in pairs(Player.Backpack:GetChildren()) do
            if v:IsA("Tool") and v.ToolTip == "Melee" then
                humanoid:EquipTool(v)
                return v.Name
            end
        end
        
        for _, v in pairs(char:GetChildren()) do
            if v:IsA("Tool") and v.ToolTip == "Melee" then
                return v.Name
            end
        end
    end
    return nil
end

-- AUTO BUSO HAKI
local function ActivateBuso()
    local char = Player.Character
    if not char then return end
    local _HasBuso = "HasBuso"
    if not char:FindFirstChild(_HasBuso) then
        pcall(function()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Buso")
        end)
    end
end

-- AUTO JUMP KHI TWEEN
local function TweenJump()
    local char = Player.Character
    if not char then return end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return end
    humanoid.Jump = true
end

-- CÁC ĐẢO TRUNG GIAN (CHỈ WORLD 3)
local IntermediateIslands = {}
if World3 then
    IntermediateIslands = {
        {name="HouseHydarIsland", pos=Vector3.new(5661.53, 1013.41, -334.96)},
        {name="Mansion", pos=Vector3.new(-12463.81, 374.95, -7550.29)},
        {name="CastleOnTheSea", pos=Vector3.new(-5047.54, 314.55, -3159.34)},
    }
end

-- HỆ THỐNG FLY
local flyEnabled = false
local flyBodyVelocity = nil
local flyBodyGyro = nil
local flyConnection = nil

local function EnableFly()
    if flyEnabled then return end
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local hrp = char.HumanoidRootPart
    if flyBodyGyro then flyBodyGyro:Destroy() end
    if flyBodyVelocity then flyBodyVelocity:Destroy() end
    
    flyBodyGyro = Instance.new("BodyGyro")
    flyBodyGyro.P = 10000
    flyBodyGyro.D = 1000
    flyBodyGyro.MaxTorque = Vector3.new(400000, 400000, 400000)
    flyBodyGyro.CFrame = hrp.CFrame
    flyBodyGyro.Parent = hrp
    
    flyBodyVelocity = Instance.new("BodyVelocity")
    flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
    flyBodyVelocity.MaxForce = Vector3.new(400000, 400000, 400000)
    flyBodyVelocity.P = 10000
    flyBodyVelocity.Parent = hrp
    
    local humanoid = char:FindFirstChild("Humanoid")
    if humanoid then humanoid.PlatformStand = true end
    
    flyEnabled = true
    
    if flyConnection then flyConnection:Disconnect() end
    flyConnection = RunService.Heartbeat:Connect(function()
        if flyEnabled and char and hrp and flyBodyGyro and flyBodyVelocity then
            flyBodyGyro.CFrame = hrp.CFrame
            flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        end
    end)
end

local function DisableFly()
    if not flyEnabled then return end
    if flyConnection then flyConnection:Disconnect() flyConnection = nil end
    if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro = nil end
    if flyBodyVelocity then flyBodyVelocity:Destroy() flyBodyVelocity = nil end
    
    local char = Player.Character
    if char then
        local humanoid = char:FindFirstChild("Humanoid")
        if humanoid then humanoid.PlatformStand = false end
    end
    flyEnabled = false
end

-- NOCLIP SYSTEM
task.spawn(function()
    while task.wait(0.01) do
        if World3 and autoBonesEnabled then
            local char = Player.Character
            if char then
                for _,v in ipairs({"HumanoidRootPart","LowerTorso","UpperTorso"}) do
                    local part = char:FindFirstChild(v)
                    if part then part.CanCollide = false end
                end
            end
        end
    end
end)

-- TWEEN SYSTEM VỚI JUMP
local currentTween = nil
local function TweenTo(targetPos)
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    if currentTween then currentTween:Cancel() end
    
    local dist = (targetPos - hrp.Position).Magnitude
    local time = dist / 350
    local tweenInfo = TweenInfo.new(time, Enum.EasingStyle.Linear)
    currentTween = TweenService:Create(hrp, tweenInfo, {CFrame = CFrame.new(targetPos)})
    currentTween:Play()
    TweenJump()
    pcall(function() currentTween.Completed:Wait() end)
end

-- TELEPORT UTILS VỚI JUMP
local function teleportSpam(position, duration)
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local t = tick() + duration
    while tick() < t do
        hrp.CFrame = CFrame.new(position + Vector3.new(0,2,0))
        TweenJump()
        task.wait()
    end
end

local function teleportMatchY(targetPos)
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    hrp.CFrame = CFrame.new(hrp.Position.X, targetPos.Y, hrp.Position.Z)
    TweenJump()
end

local function getIntermediateTeleport(a,b)
    if not World3 then return nil end
    local ad = (Vector2.new(a.X,a.Z)-Vector2.new(b.X,b.Z)).Magnitude
    local best=nil; local mb=math.huge
    for _,is in ipairs(IntermediateIslands) do
        local d = (Vector2.new(is.pos.X,is.pos.Z)-Vector2.new(b.X,b.Z)).Magnitude
        if d<mb then mb=d; best=is end
    end
    if best and mb<ad then return best end
end

-- HÀM ĐẾM BONE
local function GetBonesCount()
    local count = 0
    for _, item in pairs(Player.Backpack:GetChildren()) do
        if item.Name == "Bones" then count = count + 1 end
    end
    for _, item in pairs(Player.Character:GetChildren()) do
        if item.Name == "Bones" then count = count + 1 end
    end
    return count
end

-- BRING MOB FUNCTION
local bringMobEnabled = false
local function BringMobToTarget(targetMob, mobTypes)
    if not bringMobEnabled or not targetMob then return end
    local char = Player.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local mobNamesToBring = {}
    if type(mobTypes) == "table" then
        mobNamesToBring = mobTypes
    else
        mobNamesToBring = {mobTypes}
    end
    
    for _, mobName in pairs(mobNamesToBring) do
        for _, v in pairs(workspace.Enemies:GetChildren()) do
            if v.Name == mobName and v ~= targetMob and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 then
                local distance = (hrp.Position - v.HumanoidRootPart.Position).Magnitude
                if distance <= BringMobRadius then
                    v.HumanoidRootPart.CFrame = targetMob.HumanoidRootPart.CFrame
                end
            end
        end
    end
end

-- ATTACK FUNCTIONS
local Attack = {
    Alive = function(v) 
        return v and v:FindFirstChild("Humanoid") and v.Humanoid.Health > 0 
    end,
    Kill = function(v)
        local char = Player.Character
        if not char then return end
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        
        EquipWeapon()
        hrp.CFrame = v.HumanoidRootPart.CFrame * CFrame.new(0, 30, 0)
        TweenJump()
    end
}

-- HÀM LẤY ENEMY GẦN NHẤT
local function GetNearestEnemy(enemyNames)
    local char = Player.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    
    local nearest = nil
    local nearestDist = math.huge
    
    if type(enemyNames) == "table" then
        for _, enemyName in pairs(enemyNames) do
            for _, enemy in pairs(workspace.Enemies:GetChildren()) do
                if enemy.Name == enemyName and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                    local dist = (hrp.Position - enemy.HumanoidRootPart.Position).Magnitude
                    if dist < nearestDist then
                        nearestDist = dist
                        nearest = enemy
                    end
                end
            end
        end
    else
        for _, enemy in pairs(workspace.Enemies:GetChildren()) do
            if enemy.Name == enemyNames and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 then
                local dist = (hrp.Position - enemy.HumanoidRootPart.Position).Magnitude
                if dist < nearestDist then
                    nearestDist = dist
                    nearest = enemy
                end
            end
        end
    end
    return nearest
end

-- BIẾN AUTO BONES
local autoBonesEnabled = false
local acceptQuestsEnabled = false
local bonesLabel = nil

-- AUTO BONES LOOP
local autoBonesThread = nil
local function StartAutoBones()
    if autoBonesThread then
        task.cancel(autoBonesThread)
        autoBonesThread = nil
    end
    
    autoBonesThread = task.spawn(function()
        while task.wait(0.1) do
            if not autoBonesEnabled or not World3 then 
                DisableFly()
                continue 
            end
            
            local char = Player.Character
            if not char then continue end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp then continue end
            
            EnableFly()
            
            -- Auto Buso
            if Boud then
                ActivateBuso()
            end
            
            -- Auto equip weapon
            if autoBonesEnabled then
                EquipWeapon()
            end
            
            local questUI = PlayerGui:FindFirstChild("Main") and PlayerGui.Main:FindFirstChild("Quest")
            local BonesTable = {"Reborn Skeleton", "Living Zombie", "Demonic Soul", "Posessed Mummy"}
            local bone = GetNearestEnemy(BonesTable)
            
            if bone then
                if acceptQuestsEnabled and questUI and not questUI.Visible then
                    local questPos = CFrame.new(-9516.99316, 172.017181, 6078.46533).Position
                    TweenTo(questPos)
                    if (hrp.Position - questPos).Magnitude <= 50 then
                        local randomQuest = math.random(1, 4)
                        local questData = {
                            [1] = {"StartQuest", "HauntedQuest2", 2},
                            [2] = {"StartQuest", "HauntedQuest2", 1},
                            [3] = {"StartQuest", "HauntedQuest1", 1},
                            [4] = {"StartQuest", "HauntedQuest1", 2}
                        }
                        pcall(function()
                            ReplicatedStorage.Remotes.CommF_:InvokeServer(unpack(questData[randomQuest]))
                        end)
                    end
                end
                
                repeat
                    task.wait()
                    Attack.Kill(bone)
                    BringMobToTarget(bone, BonesTable)
                until not autoBonesEnabled or not Attack.Alive(bone) or not bone.Parent or 
                      (acceptQuestsEnabled and questUI and not questUI.Visible)
            else
                local targetPos = CFrame.new(-9495.6806640625, 453.58624267578125, 5977.3486328125).Position
                
                local island = getIntermediateTeleport(hrp.Position, targetPos)
                if island then
                    local distXZ = (Vector2.new(hrp.Position.X, hrp.Position.Z) - 
                                   Vector2.new(island.pos.X, island.pos.Z)).Magnitude
                    local delayTime = distXZ < 5000 and 0.5 or 2
                    teleportSpam(island.pos, delayTime)
                end
                
                teleportMatchY(targetPos)
                local dist = (hrp.Position - targetPos).Magnitude
                if dist > 200 then
                    TweenTo(targetPos + Vector3.new(0,2,0))
                else
                    if currentTween then currentTween:Cancel() end
                    hrp.CFrame = CFrame.new(targetPos + Vector3.new(0,2,0))
                    TweenJump()
                end
            end
        end
    end)
end

local function StopAutoBones()
    if currentTween then currentTween:Cancel() end
    autoBonesEnabled = false
    DisableFly()
    
    if autoBonesThread then
        task.cancel(autoBonesThread)
        autoBonesThread = nil
    end
end

-- ================================================
-- GUI CHÍNH
-- ================================================

-- Tạo GUI chính
local mainScreenGui = Instance.new("ScreenGui")
mainScreenGui.Name = "MainAppGUI"
mainScreenGui.DisplayOrder = 10
mainScreenGui.ResetOnSpawn = false
mainScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
mainScreenGui.Parent = PlayerGui

-- Container chính
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0.8, 0, 0.9, 0)
mainFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.ClipsDescendants = true
mainFrame.Parent = mainScreenGui

-- Tạo góc bo tròn
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0.04, 0)
uiCorner.Parent = mainFrame

-- Hiệu ứng bóng đổ
local uiShadow = Instance.new("UIStroke")
uiShadow.Color = Color3.fromRGB(0, 120, 215)
uiShadow.Thickness = 2
uiShadow.Transparency = 0.3
uiShadow.Parent = mainFrame

-- Thanh tiêu đề
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0.07, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(0, 90, 180)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0.03, 0)
titleCorner.Parent = titleBar

local titleText = Instance.new("TextLabel")
titleText.Name = "TitleText"
titleText.Size = UDim2.new(0.7, 0, 1, 0)
titleText.Position = UDim2.new(0.02, 0, 0, 0)
titleText.BackgroundTransparency = 1
titleText.Text = "⚔️ Blox Fruits GUI"
titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
titleText.TextSize = 22
titleText.Font = Enum.Font.GothamBold
titleText.TextXAlignment = Enum.TextXAlignment.Left
titleText.Parent = titleBar

-- Nút đóng
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 35, 0, 35)
closeButton.Position = UDim2.new(1, -40, 0.5, -17.5)
closeButton.AnchorPoint = Vector2.new(0, 0.5)
closeButton.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
closeButton.Text = "✕"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextSize = 20
closeButton.Font = Enum.Font.GothamBold
closeButton.Parent = titleBar

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeButton

-- Container cho tabs (có thể cuộn)
local tabsContainer = Instance.new("Frame")
tabsContainer.Name = "TabsContainer"
tabsContainer.Size = UDim2.new(1, -10, 0.1, 0)
tabsContainer.Position = UDim2.new(0.5, 0, 0.08, 0)
tabsContainer.AnchorPoint = Vector2.new(0.5, 0)
tabsContainer.BackgroundTransparency = 1
tabsContainer.ClipsDescendants = true
tabsContainer.Parent = mainFrame

-- ScrollingFrame cho tabs
local tabsScrolling = Instance.new("ScrollingFrame")
tabsScrolling.Name = "TabsScrolling"
tabsScrolling.Size = UDim2.new(1, 0, 1, 0)
tabsScrolling.Position = UDim2.new(0, 0, 0, 0)
tabsScrolling.BackgroundTransparency = 1
tabsScrolling.BorderSizePixel = 0
tabsScrolling.ScrollBarThickness = 4
tabsScrolling.ScrollBarImageColor3 = Color3.fromRGB(0, 120, 215)
tabsScrolling.VerticalScrollBarInset = Enum.ScrollBarInset.ScrollBar
tabsScrolling.ScrollingDirection = Enum.ScrollingDirection.X
tabsScrolling.Parent = tabsContainer

-- Tạo UIListLayout cho tabs
local tabsListLayout = Instance.new("UIListLayout")
tabsListLayout.FillDirection = Enum.FillDirection.Horizontal
tabsListLayout.Padding = UDim.new(0, 5)
tabsListLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabsListLayout.Parent = tabsScrolling

-- Container nội dung tab
local contentContainer = Instance.new("Frame")
contentContainer.Name = "ContentContainer"
contentContainer.Size = UDim2.new(1, -20, 0.8, -20)
contentContainer.Position = UDim2.new(0.5, 0, 0.2, 0)
contentContainer.AnchorPoint = Vector2.new(0.5, 0)
contentContainer.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
contentContainer.BackgroundTransparency = 0.1
contentContainer.Parent = mainFrame

local contentCorner = Instance.new("UICorner")
contentCorner.CornerRadius = UDim.new(0.03, 0)
contentCorner.Parent = contentContainer

-- ScrollingFrame cho nội dung
local contentScrolling = Instance.new("ScrollingFrame")
contentScrolling.Name = "ContentScrolling"
contentScrolling.Size = UDim2.new(1, -10, 1, -10)
contentScrolling.Position = UDim2.new(0.5, 0, 0.5, 0)
contentScrolling.AnchorPoint = Vector2.new(0.5, 0.5)
contentScrolling.BackgroundTransparency = 1
contentScrolling.BorderSizePixel = 0
contentScrolling.ScrollBarThickness = 6
contentScrolling.ScrollBarImageColor3 = Color3.fromRGB(0, 120, 215)
contentScrolling.CanvasSize = UDim2.new(0, 0, 0, 800)
contentScrolling.Parent = contentContainer

-- Thêm UIListLayout cho content
local contentListLayout = Instance.new("UIListLayout")
contentListLayout.Padding = UDim.new(0, 10)
contentListLayout.SortOrder = Enum.SortOrder.LayoutOrder
contentListLayout.Parent = contentScrolling

contentListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    contentScrolling.CanvasSize = UDim2.new(0, 0, 0, contentListLayout.AbsoluteContentSize.Y + 20)
end)

-- Biến lưu trữ các tab và nội dung
local tabs = {}
local tabContents = {}

-- Hàm chuyển tab
local function switchTab(tabName)
    if currentTab ~= tabName and tabContents[currentTab] then
        tabContents[currentTab].Visible = false
        if tabs[currentTab] then
            TweenService:Create(tabs[currentTab], TweenInfo.new(0.2), {
                BackgroundColor3 = Color3.fromRGB(40, 40, 60),
                TextColor3 = Color3.fromRGB(200, 200, 220)
            }):Play()
        end
    end
    
    currentTab = tabName
    if tabContents[tabName] then
        tabContents[tabName].Visible = true
    end
    
    if tabs[tabName] then
        TweenService:Create(tabs[tabName], TweenInfo.new(0.2), {
            BackgroundColor3 = Color3.fromRGB(0, 120, 215),
            TextColor3 = Color3.fromRGB(255, 255, 255)
        }):Play()
    end
    
    if tabs[tabName] then
        local tabPosition = tabs[tabName].AbsolutePosition.X - tabsScrolling.AbsolutePosition.X
        local tabWidth = tabs[tabName].AbsoluteSize.X
        
        if tabPosition < 0 then
            tabsScrolling.CanvasPosition = Vector2.new(
                math.max(0, -tabPosition + 10),
                0
            )
        elseif tabPosition + tabWidth > tabsScrolling.AbsoluteSize.X then
            tabsScrolling.CanvasPosition = Vector2.new(
                math.min(
                    tabsScrolling.CanvasSize.X.Offset - tabsScrolling.AbsoluteSize.X,
                    tabPosition + tabWidth - tabsScrolling.AbsoluteSize.X + 10
                ),
                0
            )
        end
    end
end

-- Hàm tạo nội dung tab với tích hợp Auto Bones
local function createTabContent(tabData, tabContent)
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(1, 0, 0, 40)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = tabData.Icon .. " " .. tabData.Title
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextSize = 24
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Parent = tabContent
    
    local description = Instance.new("TextLabel")
    description.Name = "Description"
    description.Size = UDim2.new(1, -20, 0, 60)
    description.Position = UDim2.new(0, 10, 0, 45)
    description.BackgroundTransparency = 1
    description.TextColor3 = Color3.fromRGB(220, 220, 220)
    description.TextSize = 16
    description.TextWrapped = true
    description.Font = Enum.Font.Gotham
    description.Parent = tabContent
    
    -- Thêm các nút điều khiển theo tab
    if tabData.Name == "Main" then
        description.Text = "Cài đặt chính và tự động hóa cơ bản"
        
        local autoFarmFrame = Instance.new("Frame")
        autoFarmFrame.Name = "AutoFarmFrame"
        autoFarmFrame.Size = UDim2.new(0.9, 0, 0, 120)
        autoFarmFrame.Position = UDim2.new(0.05, 0, 0.15, 0)
        autoFarmFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
        autoFarmFrame.BackgroundTransparency = 0.1
        autoFarmFrame.Parent = tabContent
        
        local autoFarmCorner = Instance.new("UICorner")
        autoFarmCorner.CornerRadius = UDim.new(0, 8)
        autoFarmCorner.Parent = autoFarmFrame
        
        local autoFarmTitle = Instance.new("TextLabel")
        autoFarmTitle.Size = UDim2.new(1, 0, 0, 30)
        autoFarmTitle.Position = UDim2.new(0, 0, 0, 5)
        autoFarmTitle.BackgroundTransparency = 1
        autoFarmTitle.Text = "⚔️ Auto Farm Settings"
        autoFarmTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        autoFarmTitle.TextSize = 18
        autoFarmTitle.Font = Enum.Font.GothamBold
        autoFarmTitle.Parent = autoFarmFrame
        
        -- Auto Farm Level Button
        local button1 = Instance.new("TextButton")
        button1.Name = "AutoFarmButton"
        button1.Size = UDim2.new(0.9, 0, 0, 40)
        button1.Position = UDim2.new(0.05, 0, 0.3, 0)
        button1.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
        button1.Text = "⚔️ Auto Farm Level"
        button1.TextColor3 = Color3.fromRGB(255, 255, 255)
        button1.TextSize = 16
        button1.Font = Enum.Font.GothamBold
        button1.Parent = autoFarmFrame
        
        local button1Corner = Instance.new("UICorner")
        button1Corner.CornerRadius = UDim.new(0, 6)
        button1Corner.Parent = button1
        
        -- Auto Farm Status Label
        local autoFarmStatus = Instance.new("TextLabel")
        autoFarmStatus.Size = UDim2.new(0.9, 0, 0, 25)
        autoFarmStatus.Position = UDim2.new(0.05, 0, 0.7, 0)
        autoFarmStatus.BackgroundTransparency = 1
        autoFarmStatus.Text = "Trạng thái: TẮT"
        autoFarmStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
        autoFarmStatus.TextSize = 14
        autoFarmStatus.Font = Enum.Font.Gotham
        autoFarmStatus.Parent = autoFarmFrame
        
        button1.MouseButton1Click:Connect(function()
            if autoFarmStatus.Text == "Trạng thái: TẮT" then
                autoFarmStatus.Text = "Trạng thái: BẬT"
                autoFarmStatus.TextColor3 = Color3.fromRGB(100, 255, 100)
                print("Auto Farm Level đã BẬT")
            else
                autoFarmStatus.Text = "Trạng thái: TẮT"
                autoFarmStatus.TextColor3 = Color3.fromRGB(255, 100, 100)
                print("Auto Farm Level đã TẮT")
            end
        end)
        
        -- AUTO BONES SECTION (CHỈ HIỂN THỊ NẾU LÀ WORLD 3)
        if World3 then
            local bonesFrame = Instance.new("Frame")
            bonesFrame.Name = "BonesFrame"
            bonesFrame.Size = UDim2.new(0.9, 0, 0, 200)
            bonesFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
            bonesFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
            bonesFrame.BackgroundTransparency = 0.1
            bonesFrame.Parent = tabContent
            
            local bonesCorner = Instance.new("UICorner")
            bonesCorner.CornerRadius = UDim.new(0, 8)
            bonesCorner.Parent = bonesFrame
            
            local bonesTitle = Instance.new("TextLabel")
            bonesTitle.Size = UDim2.new(1, 0, 0, 30)
            bonesTitle.Position = UDim2.new(0, 0, 0, 5)
            bonesTitle.BackgroundTransparency = 1
            bonesTitle.Text = "💀 Auto Farm Bones (World 3)"
            bonesTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
            bonesTitle.TextSize = 18
            bonesTitle.Font = Enum.Font.GothamBold
            bonesTitle.Parent = bonesFrame
            
            -- World Info
            local worldInfo = Instance.new("TextLabel")
            worldInfo.Size = UDim2.new(0.9, 0, 0, 20)
            worldInfo.Position = UDim2.new(0.05, 0, 0.15, 0)
            worldInfo.BackgroundTransparency = 1
            worldInfo.Text = "World: " .. worldName
            worldInfo.TextColor3 = Color3.fromRGB(200, 200, 255)
            worldInfo.TextSize = 14
            worldInfo.Font = Enum.Font.Gotham
            worldInfo.Parent = bonesFrame
            
            -- Auto Bones Button
            local autoBonesBtn = Instance.new("TextButton")
            autoBonesBtn.Name = "AutoBonesButton"
            autoBonesBtn.Size = UDim2.new(0.9, 0, 0, 35)
            autoBonesBtn.Position = UDim2.new(0.05, 0, 0.3, 0)
            autoBonesBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
            autoBonesBtn.Text = "💀 Auto Bones: OFF"
            autoBonesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            autoBonesBtn.TextSize = 15
            autoBonesBtn.Font = Enum.Font.GothamBold
            autoBonesBtn.Parent = bonesFrame
            
            local autoBonesCorner = Instance.new("UICorner")
            autoBonesCorner.CornerRadius = UDim.new(0, 6)
            autoBonesCorner.Parent = autoBonesBtn
            
            -- Bring Mob Button
            local bringMobBtn = Instance.new("TextButton")
            bringMobBtn.Name = "BringMobButton"
            bringMobBtn.Size = UDim2.new(0.9, 0, 0, 35)
            bringMobBtn.Position = UDim2.new(0.05, 0, 0.5, 0)
            bringMobBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
            bringMobBtn.Text = "🌀 Bring Mob: OFF"
            bringMobBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            bringMobBtn.TextSize = 15
            bringMobBtn.Font = Enum.Font.GothamBold
            bringMobBtn.Parent = bonesFrame
            
            local bringMobCorner = Instance.new("UICorner")
            bringMobCorner.CornerRadius = UDim.new(0, 6)
            bringMobCorner.Parent = bringMobBtn
            
            -- Auto Quest Button
            local autoQuestBtn = Instance.new("TextButton")
            autoQuestBtn.Name = "AutoQuestButton"
            autoQuestBtn.Size = UDim2.new(0.9, 0, 0, 35)
            autoQuestBtn.Position = UDim2.new(0.05, 0, 0.7, 0)
            autoQuestBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
            autoQuestBtn.Text = "📜 Auto Quest: OFF"
            autoQuestBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            autoQuestBtn.TextSize = 15
            autoQuestBtn.Font = Enum.Font.GothamBold
            autoQuestBtn.Parent = bonesFrame
            
            local autoQuestCorner = Instance.new("UICorner")
            autoQuestCorner.CornerRadius = UDim.new(0, 6)
            autoQuestCorner.Parent = autoQuestBtn
            
            -- Bones Counter
            local bonesCounter = Instance.new("TextLabel")
            bonesCounter.Name = "BonesCounter"
            bonesCounter.Size = UDim2.new(0.9, 0, 0, 20)
            bonesCounter.Position = UDim2.new(0.05, 0, 0.9, 0)
            bonesCounter.BackgroundTransparency = 1
            bonesCounter.Text = "💀 Bones: 0"
            bonesCounter.TextColor3 = Color3.fromRGB(255, 255, 100)
            bonesCounter.TextSize = 14
            bonesCounter.Font = Enum.Font.GothamBold
            bonesCounter.Parent = bonesFrame
            
            bonesLabel = bonesCounter
            
            -- Status Label
            local statusLabel = Instance.new("TextLabel")
            statusLabel.Size = UDim2.new(0.9, 0, 0, 20)
            statusLabel.Position = UDim2.new(0.05, 0, 0.85, 0)
            statusLabel.BackgroundTransparency = 1
            statusLabel.Text = "Status: Ready"
            statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
            statusLabel.TextSize = 12
            statusLabel.Font = Enum.Font.Gotham
            statusLabel.Parent = bonesFrame
            
            -- Control Buttons
            local controlFrame = Instance.new("Frame")
            controlFrame.Size = UDim2.new(0.9, 0, 0, 35)
            controlFrame.Position = UDim2.new(0.05, 0, 0.8, 0)
            controlFrame.BackgroundTransparency = 1
            controlFrame.Parent = bonesFrame
            
            local stopBtn = Instance.new("TextButton")
            stopBtn.Size = UDim2.new(0.48, 0, 1, 0)
            stopBtn.Position = UDim2.new(0, 0, 0, 0)
            stopBtn.BackgroundColor3 = Color3.fromRGB(220, 60, 60)
            stopBtn.Text = "🛑 STOP"
            stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            stopBtn.TextSize = 14
            stopBtn.Font = Enum.Font.GothamBold
            stopBtn.Parent = controlFrame
            
            local stopCorner = Instance.new("UICorner")
            stopCorner.CornerRadius = UDim.new(0, 6)
            stopCorner.Parent = stopBtn
            
            local flyBtn = Instance.new("TextButton")
            flyBtn.Size = UDim2.new(0.48, 0, 1, 0)
            flyBtn.Position = UDim2.new(0.52, 0, 0, 0)
            flyBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 220)
            flyBtn.Text = "🦅 Fly: OFF"
            flyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
            flyBtn.TextSize = 14
            flyBtn.Font = Enum.Font.GothamBold
            flyBtn.Parent = controlFrame
            
            local flyCorner = Instance.new("UICorner")
            flyCorner.CornerRadius = UDim.new(0, 6)
            flyCorner.Parent = flyBtn
            
            -- Button Events
            autoBonesBtn.MouseButton1Click:Connect(function()
                autoBonesEnabled = not autoBonesEnabled
                if autoBonesEnabled then
                    autoBonesBtn.Text = "💀 Auto Bones: ON"
                    autoBonesBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                    statusLabel.Text = "Status: Farming Bones..."
                    StartAutoBones()
                else
                    autoBonesBtn.Text = "💀 Auto Bones: OFF"
                    autoBonesBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
                    statusLabel.Text = "Status: Stopped"
                    StopAutoBones()
                end
            end)
            
            bringMobBtn.MouseButton1Click:Connect(function()
                bringMobEnabled = not bringMobEnabled
                if bringMobEnabled then
                    bringMobBtn.Text = "🌀 Bring Mob: ON"
                    bringMobBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                else
                    bringMobBtn.Text = "🌀 Bring Mob: OFF"
                    bringMobBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
                end
            end)
            
            autoQuestBtn.MouseButton1Click:Connect(function()
                acceptQuestsEnabled = not acceptQuestsEnabled
                if acceptQuestsEnabled then
                    autoQuestBtn.Text = "📜 Auto Quest: ON"
                    autoQuestBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                else
                    autoQuestBtn.Text = "📜 Auto Quest: OFF"
                    autoQuestBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
                end
            end)
            
            stopBtn.MouseButton1Click:Connect(function()
                StopAutoBones()
                autoBonesEnabled = false
                autoBonesBtn.Text = "💀 Auto Bones: OFF"
                autoBonesBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
                statusLabel.Text = "Status: Stopped"
                bringMobEnabled = false
                bringMobBtn.Text = "🌀 Bring Mob: OFF"
                bringMobBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
            end)
            
            flyBtn.MouseButton1Click:Connect(function()
                if flyEnabled then
                    DisableFly()
                    flyBtn.Text = "🦅 Fly: OFF"
                    flyBtn.BackgroundColor3 = Color3.fromRGB(60, 180, 220)
                else
                    EnableFly()
                    flyBtn.Text = "🦅 Fly: ON"
                    flyBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
                end
            end)
        else
            -- Hiển thị thông báo nếu không phải World 3
            local noWorld3Frame = Instance.new("Frame")
            noWorld3Frame.Size = UDim2.new(0.9, 0, 0, 100)
            noWorld3Frame.Position = UDim2.new(0.05, 0, 0.4, 0)
            noWorld3Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
            noWorld3Frame.BackgroundTransparency = 0.1
            noWorld3Frame.Parent = tabContent
            
            local noWorld3Corner = Instance.new("UICorner")
            noWorld3Corner.CornerRadius = UDim.new(0, 8)
            noWorld3Corner.Parent = noWorld3Frame
            
            local noWorld3Label = Instance.new("TextLabel")
            noWorld3Label.Size = UDim2.new(0.9, 0, 0.8, 0)
            noWorld3Label.Position = UDim2.new(0.05, 0, 0.1, 0)
            noWorld3Label.BackgroundTransparency = 1
            noWorld3Label.Text = "⚠️ Auto Farm Bones chỉ hoạt động ở World 3\n\nHiện tại bạn đang ở:\n" .. worldName
            noWorld3Label.TextColor3 = Color3.fromRGB(255, 200, 100)
            noWorld3Label.TextSize = 14
            noWorld3Label.TextWrapped = true
            noWorld3Label.Font = Enum.Font.Gotham
            noWorld3Label.Parent = noWorld3Frame
        end
        
    elseif tabData.Name == "Travel" then
        description.Text = "Di chuyển nhanh đến các đảo và vị trí quan trọng"
        
        local teleportLabel = Instance.new("TextLabel")
        teleportLabel.Size = UDim2.new(1, 0, 0, 30)
        teleportLabel.Position = UDim2.new(0, 0, 0.15, 0)
        teleportLabel.BackgroundTransparency = 1
        teleportLabel.Text = "📍 Danh sách địa điểm:"
        teleportLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        teleportLabel.TextSize = 20
        teleportLabel.Font = Enum.Font.GothamBold
        teleportLabel.Parent = tabContent
        
        -- Thêm các nút teleport
        local teleports = {"Marine Base", "Pirate Village", "Jungle", "Desert", "Snow Village"}
        for i, location in ipairs(teleports) do
            local teleportButton = Instance.new("TextButton")
            teleportButton.Size = UDim2.new(0.9, 0, 0, 40)
            teleportButton.Position = UDim2.new(0.05, 0, 0.2 + (i * 0.1), 0)
            teleportButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            teleportButton.Text = "📍 " .. location
            teleportButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            teleportButton.TextSize = 16
            teleportButton.Font = Enum.Font.Gotham
            teleportButton.Parent = tabContent
            
            local teleportCorner = Instance.new("UICorner")
            teleportCorner.CornerRadius = UDim.new(0, 6)
            teleportCorner.Parent = teleportButton
            
            teleportButton.MouseButton1Click:Connect(function()
                print("Di chuyển đến: " .. location)
            end)
        end
        
    elseif tabData.Name == "Combat" then
        description.Text = "Cài đặt chiến đấu và PVP"
        
        local toggle1 = Instance.new("TextButton")
        toggle1.Size = UDim2.new(0.9, 0, 0, 40)
        toggle1.Position = UDim2.new(0.05, 0, 0.2, 0)
        toggle1.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        toggle1.Text = "⚔️ Auto Attack"
        toggle1.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggle1.TextSize = 16
        toggle1.Font = Enum.Font.Gotham
        toggle1.Parent = tabContent
        
        local toggle1Corner = Instance.new("UICorner")
        toggle1Corner.CornerRadius = UDim.new(0, 6)
        toggle1Corner.Parent = toggle1
        
        local toggle2 = Instance.new("TextButton")
        toggle2.Size = UDim2.new(0.9, 0, 0, 40)
        toggle2.Position = UDim2.new(0.05, 0, 0.3, 0)
        toggle2.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
        toggle2.Text = "🛡️ Auto Block"
        toggle2.TextColor3 = Color3.fromRGB(255, 255, 255)
        toggle2.TextSize = 16
        toggle2.Font = Enum.Font.Gotham
        toggle2.Parent = tabContent
        
        local toggle2Corner = Instance.new("UICorner")
        toggle2Corner.CornerRadius = UDim.new(0, 6)
        toggle2Corner.Parent = toggle2
        
    elseif tabData.Name == "Fruit" then
        description.Text = "Quản lý và sử dụng Trái ác quỷ"
        
        local fruitLabel = Instance.new("TextLabel")
        fruitLabel.Size = UDim2.new(1, 0, 0, 30)
        fruitLabel.Position = UDim2.new(0, 0, 0.15, 0)
        fruitLabel.BackgroundTransparency = 1
        fruitLabel.Text = "🍎 Quản lý Trái ác quỷ:"
        fruitLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        fruitLabel.TextSize = 20
        fruitLabel.Font = Enum.Font.GothamBold
        fruitLabel.Parent = tabContent
        
        local fruits = {"Light", "Dark", "Flame", "Ice", "Quake"}
        for i, fruit in ipairs(fruits) do
            local fruitButton = Instance.new("TextButton")
            fruitButton.Size = UDim2.new(0.9, 0, 0, 40)
            fruitButton.Position = UDim2.new(0.05, 0, 0.2 + (i * 0.1), 0)
            fruitButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            fruitButton.Text = "🍎 " .. fruit
            fruitButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            fruitButton.TextSize = 16
            fruitButton.Font = Enum.Font.Gotham
            fruitButton.Parent = tabContent
        end
        
    elseif tabData.Name == "Shop" then
        description.Text = "Mua sắm vũ khí, phụ kiện và vật phẩm"
        
        local shopLabel = Instance.new("TextLabel")
        shopLabel.Size = UDim2.new(1, 0, 0, 30)
        shopLabel.Position = UDim2.new(0, 0, 0.15, 0)
        shopLabel.BackgroundTransparency = 1
        shopLabel.Text = "🛒 Cửa hàng:"
        shopLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        shopLabel.TextSize = 20
        shopLabel.Font = Enum.Font.GothamBold
        shopLabel.Parent = tabContent
        
        local items = {"Swords", "Guns", "Fruits", "Accessories"}
        for i, item in ipairs(items) do
            local itemButton = Instance.new("TextButton")
            itemButton.Size = UDim2.new(0.9, 0, 0, 40)
            itemButton.Position = UDim2.new(0.05, 0, 0.2 + (i * 0.1), 0)
            itemButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            itemButton.Text = "💰 Mua " .. item
            itemButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            itemButton.TextSize = 16
            itemButton.Font = Enum.Font.Gotham
            itemButton.Parent = tabContent
        end
        
    elseif tabData.Name == "Settings" then
        description.Text = "Cài đặt hệ thống và tùy chọn"
        
        local settingsFrame = Instance.new("Frame")
        settingsFrame.Size = UDim2.new(0.9, 0, 0, 150)
        settingsFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
        settingsFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
        settingsFrame.BackgroundTransparency = 0.1
        settingsFrame.Parent = tabContent
        
        local settingsCorner = Instance.new("UICorner")
        settingsCorner.CornerRadius = UDim.new(0, 8)
        settingsCorner.Parent = settingsFrame
        
        local settingsTitle = Instance.new("TextLabel")
        settingsTitle.Size = UDim2.new(1, 0, 0, 30)
        settingsTitle.Position = UDim2.new(0, 0, 0, 5)
        settingsTitle.BackgroundTransparency = 1
        settingsTitle.Text = "⚙️ System Settings"
        settingsTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
        settingsTitle.TextSize = 18
        settingsTitle.Font = Enum.Font.GothamBold
        settingsTitle.Parent = settingsFrame
        
        -- Join Marines Button
        local joinMarinesBtn = Instance.new("TextButton")
        joinMarinesBtn.Size = UDim2.new(0.9, 0, 0, 35)
        joinMarinesBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
        joinMarinesBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
        joinMarinesBtn.Text = "💂 Join Marines"
        joinMarinesBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        joinMarinesBtn.TextSize = 15
        joinMarinesBtn.Font = Enum.Font.GothamBold
        joinMarinesBtn.Parent = settingsFrame
        
        local joinMarinesCorner = Instance.new("UICorner")
        joinMarinesCorner.CornerRadius = UDim.new(0, 6)
        joinMarinesCorner.Parent = joinMarinesBtn
        
        -- Auto Buso Toggle
        local autoBusoBtn = Instance.new("TextButton")
        autoBusoBtn.Size = UDim2.new(0.9, 0, 0, 35)
        autoBusoBtn.Position = UDim2.new(0.05, 0, 0.4, 0)
        autoBusoBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
        autoBusoBtn.Text = "🛡️ Auto Buso: ON"
        autoBusoBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        autoBusoBtn.TextSize = 15
        autoBusoBtn.Font = Enum.Font.GothamBold
        autoBusoBtn.Parent = settingsFrame
        
        local autoBusoCorner = Instance.new("UICorner")
        autoBusoCorner.CornerRadius = UDim.new(0, 6)
        autoBusoCorner.Parent = autoBusoBtn
        
        -- Noclip Toggle
        local noclipBtn = Instance.new("TextButton")
        noclipBtn.Size = UDim2.new(0.9, 0, 0, 35)
        noclipBtn.Position = UDim2.new(0.05, 0, 0.6, 0)
        noclipBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
        noclipBtn.Text = "👻 Noclip: OFF"
        noclipBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        noclipBtn.TextSize = 15
        noclipBtn.Font = Enum.Font.GothamBold
        noclipBtn.Parent = settingsFrame
        
        local noclipCorner = Instance.new("UICorner")
        noclipCorner.CornerRadius = UDim.new(0, 6)
        noclipCorner.Parent = noclipBtn
        
        -- Button Events
        joinMarinesBtn.MouseButton1Click:Connect(function()
            JoinMarines()
            joinMarinesBtn.Text = "✅ Marines Joined"
            joinMarinesBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        end)
        
        autoBusoBtn.MouseButton1Click:Connect(function()
            Boud = not Boud
            if Boud then
                autoBusoBtn.Text = "🛡️ Auto Buso: ON"
                autoBusoBtn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
            else
                autoBusoBtn.Text = "🛡️ Auto Buso: OFF"
                autoBusoBtn.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
            end
        end)
        
    else
        description.Text = "Nội dung cho " .. tabData.Title
    end
end

-- Hàm tạo tab
local function createTab(tabData)
    local tabButton = Instance.new("TextButton")
    tabButton.Name = "Tab_" .. tabData.Name
    tabButton.Size = UDim2.new(0, isMobile and 120 or 110, 1, -5)
    tabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    tabButton.Text = tabData.Icon .. " " .. tabData.Title
    tabButton.TextColor3 = Color3.fromRGB(200, 200, 220)
    tabButton.TextSize = isMobile and 14 or 13
    tabButton.Font = Enum.Font.Gotham
    tabButton.AutoButtonColor = true
    tabButton.LayoutOrder = #tabs + 1
    
    local tabCorner = Instance.new("UICorner")
    tabCorner.CornerRadius = UDim.new(0.1, 0)
    tabCorner.Parent = tabButton
    
    local tabStroke = Instance.new("UIStroke")
    tabStroke.Color = Color3.fromRGB(0, 120, 215)
    tabStroke.Thickness = 2
    tabStroke.Transparency = 0.5
    tabStroke.Parent = tabButton
    
    tabButton.Parent = tabsScrolling
    
    local tabContent = Instance.new("Frame")
    tabContent.Name = "Content_" .. tabData.Name
    tabContent.Size = UDim2.new(1, 0, 0, 600)
    tabContent.BackgroundTransparency = 1
    tabContent.Visible = false
    tabContent.LayoutOrder = #tabs + 1
    tabContent.Parent = contentScrolling
    
    createTabContent(tabData, tabContent)
    
    tabButton.MouseButton1Click:Connect(function()
        switchTab(tabData.Name)
    end)
    
    if isMobile then
        local function onTouch()
            switchTab(tabData.Name)
        end
        
        tabButton.TouchTap:Connect(onTouch)
        tabButton.TouchLongPress:Connect(onTouch)
    end
    
    if not isMobile then
        tabButton.MouseEnter:Connect(function()
            if currentTab ~= tabData.Name then
                TweenService:Create(tabButton, TweenInfo.new(0.2), {
                    BackgroundColor3 = Color3.fromRGB(60, 60, 90)
                }):Play()
            end
        end)
        
        tabButton.MouseLeave:Connect(function()
            if currentTab ~= tabData.Name then
                TweenService:Create(tabButton, TweenInfo.new(0.2), {
                    BackgroundColor3 = Color3.fromRGB(40, 40, 60)
                }):Play()
            end
        end)
    end
    
    tabs[tabData.Name] = tabButton
    tabContents[tabData.Name] = tabContent
    
    return tabButton, tabContent
end

-- Tạo tất cả các tab
for i, tabData in ipairs(tabsData) do
    createTab(tabData)
end

-- Tính toán kích thước cho ScrollingFrame tabs
local function updateTabsSize()
    local totalWidth = 0
    for _, tab in pairs(tabs) do
        totalWidth = totalWidth + tab.AbsoluteSize.X + 5
    end
    tabsScrolling.CanvasSize = UDim2.new(0, totalWidth, 0, 0)
end

task.wait(0.1)
updateTabsSize()

-- Tạo nút luôn hiển thị (Always-on-top button) - CÓ THỂ KÉO
local toggleScreenGui = Instance.new("ScreenGui")
toggleScreenGui.Name = "ToggleButtonGUI"
toggleScreenGui.DisplayOrder = 999
toggleScreenGui.ResetOnSpawn = false
toggleScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
toggleScreenGui.Parent = PlayerGui

local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = isMobile and UDim2.new(0, 60, 0, 60) or UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(1, -70, 1, -70)
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 120, 215)
toggleButton.Text = "⚙️"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextSize = isMobile and 30 or 24
toggleButton.Font = Enum.Font.GothamBold
toggleButton.AutoButtonColor = true
toggleButton.Parent = toggleScreenGui

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = isMobile and UDim.new(0, 30) or UDim.new(0, 25)
toggleCorner.Parent = toggleButton

local toggleShadow = Instance.new("UIStroke")
toggleShadow.Color = Color3.fromRGB(255, 255, 255)
toggleShadow.Thickness = 2
toggleShadow.Transparency = 0.5
toggleShadow.Parent = toggleButton

-- FUNCTIONS
local function toggleGUI()
    isGUIVisible = not isGUIVisible
    
    if isGUIVisible then
        mainScreenGui.Enabled = true
        mainFrame.Visible = true
        
        TweenService:Create(
            mainFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = UDim2.new(0.5, 0, 0.5, 0), Size = UDim2.new(0.8, 0, 0.9, 0)}
        ):Play()
        
        toggleButton.Text = "⚙️"
    else
        TweenService:Create(
            mainFrame,
            TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
            {Position = UDim2.new(0.5, 0, 1.5, 0), Size = UDim2.new(0.8, 0, 0, 0)}
        ):Play()
        
        toggleButton.Text = "👁️"
        
        task.wait(0.3)
        mainFrame.Visible = false
    end
end

-- KÉO THẢ CHO MAIN GUI
local draggingMain = false
local dragInputMain, dragStartMain, startPosMain

local function updateMain(input)
    local delta = input.Position - dragStartMain
    mainFrame.Position = UDim2.new(
        startPosMain.X.Scale, 
        startPosMain.X.Offset + delta.X,
        startPosMain.Y.Scale, 
        startPosMain.Y.Offset + delta.Y
    )
end

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingMain = true
        dragStartMain = input.Position
        startPosMain = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                draggingMain = false
            end
        end)
    end
end)

titleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInputMain = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingMain and (input == dragInputMain) then
        updateMain(input)
    end
end)

-- KÉO THẢ CHO NÚT TOGGLE
local draggingToggle = false
local dragInputToggle, dragStartToggle, startPosToggle

local function updateToggle(input)
    local delta = input.Position - dragStartToggle
    local viewportSize = workspace.CurrentCamera.ViewportSize
    
    local newX = startPosToggle.X.Offset + delta.X
    local newY = startPosToggle.Y.Offset + delta.Y
    
    newX = math.clamp(newX, 0, viewportSize.X - toggleButton.AbsoluteSize.X)
    newY = math.clamp(newY, 0, viewportSize.Y - toggleButton.AbsoluteSize.Y)
    
    toggleButton.Position = UDim2.new(0, newX, 0, newY)
end

local longPressDetected = false
local longPressThread

toggleButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        longPressDetected = false
        local pressStartTime = tick()
        
        longPressThread = task.spawn(function()
            task.wait(0.5)
            if tick() - pressStartTime >= 0.5 then
                longPressDetected = true
                draggingToggle = true
                dragStartToggle = input.Position
                startPosToggle = toggleButton.Position
                
                TweenService:Create(
                    toggleButton,
                    TweenInfo.new(0.2),
                    {BackgroundColor3 = Color3.fromRGB(0, 180, 255)}
                ):Play()
            end
        end)
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                if longPressThread then
                    task.cancel(longPressThread)
                end
                
                if not longPressDetected then
                    toggleGUI()
                else
                    draggingToggle = false
                    TweenService:Create(
                        toggleButton,
                        TweenInfo.new(0.2),
                        {BackgroundColor3 = Color3.fromRGB(0, 120, 215)}
                    ):Play()
                end
                longPressDetected = false
            end
        end)
    end
end)

toggleButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInputToggle = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingToggle and (input == dragInputToggle) then
        updateToggle(input)
    end
end)

-- Kết nối sự kiện
closeButton.MouseButton1Click:Connect(toggleGUI)

if isMobile then
    closeButton.TouchTap:Connect(toggleGUI)
end

-- Hiệu ứng mobile cho nút toggle
if isMobile then
    toggleButton.TouchTap:Connect(function()
        if not longPressDetected and not draggingToggle then
            local originalSize = toggleButton.Size
            TweenService:Create(
                toggleButton,
                TweenInfo.new(0.1),
                {Size = originalSize - UDim2.new(0, 10, 0, 10)}
            ):Play()
            
            task.wait(0.1)
            
            TweenService:Create(
                toggleButton,
                TweenInfo.new(0.1),
                {Size = originalSize}
            ):Play()
        end
    end)
end

-- Tự động điều chỉnh cho mobile
if isMobile then
    local function updateMobileLayout()
        local viewportSize = workspace.CurrentCamera.ViewportSize
        if viewportSize.Y < 700 then
            mainFrame.Size = UDim2.new(0.95, 0, 0.95, 0)
        else
            mainFrame.Size = UDim2.new(0.8, 0, 0.9, 0)
        end
    end
    
    workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(updateMobileLayout)
    updateMobileLayout()
end

-- CẬP NHẬT BONES COUNTER
task.spawn(function()
    while task.wait(0.5) do
        if bonesLabel then
            local boneCount = GetBonesCount()
            bonesLabel.Text = "💀 Bones: " .. boneCount
        end
    end
end)

-- Mặc định chọn tab đầu tiên
task.wait(0.2)
JoinMarines()
switchTab("Main")

-- Thông báo
print("═══════════════════════════════════════════════════")
print("✅ Blox Fruits GUI + Auto Farm Bones đã sẵn sàng!")
print("• World: " .. worldName .. (World3 and " (Auto Bones Enabled)" or ""))
print("• Marines Auto Join: " .. (HasJoinedMarines and "Enabled" or "Disabled"))
print("• Tab hiện tại: " .. currentTab)
print("═══════════════════════════════════════════════════")
