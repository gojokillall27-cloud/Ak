-- Sigma Hub - Jujutsu Shenanigans (WindUI Blue Theme)

local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

-- Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local CoreGui = game:GetService("CoreGui")

-- Global Variables
local speedEnabled, speedConnection = false, nil
local noclipEnabled, noclipConnection = false, nil
local auto02sUltEnabled, gojoLoopConnection = false, nil
local esp2DEnabled, skeletonEnabled = false, false
local invisibleEnabled = false
local originalTransparencies = {}

-- Bypass AC Variables
local bypassACEnabled = false
local bypassACThread = nil

-- Fun Tab Variables
local socLoDoing = false
local socLoAnimTrack = nil
local socLoThread = nil

-- Orbit Variables
local orbitEnabled = false
local orbitTargetPlayer = nil
local orbitSpeed = 20
local orbitDistance = 5
local orbitAngle = 0
local orbitConnection = nil

local function GetClosestTarget()
    local closest, minDist = nil, math.huge
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end
    local myPos = myChar.HumanoidRootPart.Position

    for _, v in ipairs(Players:GetPlayers()) do
        if v ~= LocalPlayer and v.Character and v.Character:FindFirstChild("HumanoidRootPart") and v.Character:FindFirstChild("Humanoid") then
            if v.Character.Humanoid.Health > 0 then
                local dist = (v.Character.HumanoidRootPart.Position - myPos).Magnitude
                if dist < minDist then
                    minDist = dist
                    closest = v
                end
            end
        end
    end
    return closest
end

-- Hàm xử lý Auto Bypass AC
local function StartAutoBypassAC(bypassToggleElement)
    local targetPos = Vector3.new(-120, 210, -200)

    bypassACThread = task.spawn(function()
        while bypassACEnabled do
            local char = LocalPlayer.Character
            if not char or not char:FindFirstChild("HumanoidRootPart") or not char:FindFirstChild("Humanoid") then
                task.wait(1)
                continue
            end

            if char.Humanoid.Health <= 0 then
                LocalPlayer.CharacterAdded:Wait()
                task.wait(1)
                continue
            end

            local hrp = char.HumanoidRootPart
            local oldPos = hrp.Position
            
            -- Thực hiện dịch chuyển
            hrp.CFrame = CFrame.new(targetPos)
            WindUI:Notify({ Title = "Bypass AC", Content = "Đang thử nghiệm dịch chuyển Bypass...", Duration = 2 })

            task.wait(0.3)

            if not bypassACEnabled then break end

            if char and char:FindFirstChild("HumanoidRootPart") then
                local currentPos = char.HumanoidRootPart.Position
                -- Thất bại: Bị Anti-Cheat kéo lại vị trí cũ
                if (currentPos - oldPos).Magnitude < 10 and (targetPos - oldPos).Magnitude > 30 then
                    WindUI:Notify({ Title = "Bypass AC", Content = "Thất bại (Phát hiện Anti-Cheat)! Đang Reset và thử lại...", Duration = 3 })
                    
                    pcall(function()
                        ReplicatedStorage.Knit.Knit.Services.JoinService.RE.Reset:FireServer()
                    end)
                    
                    -- Chờ hồi sinh để thử lại vòng lặp tiếp theo
                    LocalPlayer.CharacterAdded:Wait()
                    task.wait(1.5)
                else
                    -- Thành công: Không bị kéo về
                    WindUI:Notify({ Title = "Bypass AC", Content = "Bypass Anti-Cheat THÀNH CÔNG!", Duration = 4 })
                    bypassACEnabled = false
                    if bypassToggleElement then
                        bypassToggleElement:Set(false)
                    end
                    break
                end
            end
        end
    end)
end

----------------------------------------------------
-- WINDUI WINDOW INITIALIZATION (SIGMA HUB - BLUE THEME)
----------------------------------------------------
local Window = WindUI:CreateWindow({
    Title = "Sigma Hub - JJS",
    Icon = "rbxassetid://10723415649",
    Author = "Sigma",
    Folder = "SigmaHubJJS",
    Size = UDim2.fromOffset(520, 340),
    Transparent = true,
    Theme = "Dark",
    SideBarWidth = 140,
    HasOutline = true
})

WindUI:AddTheme({
    Name = "AnimeBlueTheme",
    Accent = Color3.fromRGB(30, 115, 235),
    Outline = Color3.fromRGB(90, 180, 255),
    Text = Color3.fromRGB(255, 255, 255),
    PlaceholderText = Color3.fromRGB(160, 210, 255),
    Background = Color3.fromRGB(15, 22, 35)
})
WindUI:SetTheme("AnimeBlueTheme")

----------------------------------------------------
-- TOGGLE MENU BUTTON
----------------------------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SigmaMainGuiHolder"
ScreenGui.Parent = (gethui and gethui()) or CoreGui or LocalPlayer:WaitForChild("PlayerGui")

local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Name = "ToggleMenuBtn"
ToggleBtn.Parent = ScreenGui
ToggleBtn.BackgroundColor3 = Color3.fromRGB(30, 115, 235)
ToggleBtn.Position = UDim2.new(0.05, 0, 0.2, 0)
ToggleBtn.Size = UDim2.new(0, 45, 0, 45)
ToggleBtn.Text = "GUI"
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 16
ToggleBtn.Font = Enum.Font.SourceSansBold
ToggleBtn.Active = true
ToggleBtn.Draggable = true

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(1, 0)
UICorner.Parent = ToggleBtn

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 2
UIStroke.Color = Color3.fromRGB(90, 180, 255)
UIStroke.Parent = ToggleBtn

ToggleBtn.MouseButton1Click:Connect(function()
    Window:Toggle()
end)

----------------------------------------------------
-- TABS INITIALIZATION
----------------------------------------------------
local MainTab = Window:Tab({ Title = "Main", Icon = "home" })
local CharactersTab = Window:Tab({ Title = "Characters", Icon = "user" })
local CombatTab = Window:Tab({ Title = "Combat", Icon = "sword" })
local TPTab = Window:Tab({ Title = "TP Locations", Icon = "map-pin" })
local ESPTab = Window:Tab({ Title = "ESP", Icon = "eye" })
local FunTab = Window:Tab({ Title = "Fun", Icon = "smile" })
local MiscTab = Window:Tab({ Title = "Misc", Icon = "more-horizontal" })

----------------------------------------------------
-- MAIN TAB
----------------------------------------------------
MainTab:Toggle({
    Title = "Speed",
    Default = false,
    Callback = function(Value)
        speedEnabled = Value
        if speedEnabled then
            speedConnection = RunService.RenderStepped:Connect(function()
                local char = LocalPlayer.Character
                if char and char:FindFirstChild("Humanoid") and char:FindFirstChild("HumanoidRootPart") then
                    if char.Humanoid.MoveDirection.Magnitude > 0 then
                        char:TranslateBy(char.Humanoid.MoveDirection * 0.8)
                    end
                end
            end)
        else
            if speedConnection then speedConnection:Disconnect() end
        end
    end
})

MainTab:Toggle({
    Title = "Noclip",
    Default = false,
    Callback = function(Value)
        noclipEnabled = Value
        if noclipEnabled then
            noclipConnection = RunService.Stepped:Connect(function()
                if LocalPlayer.Character then
                    for _, part in pairs(LocalPlayer.Character:GetDescendants()) do
                        if part:IsA("BasePart") then part.CanCollide = false end
                    end
                end
            end)
        else
            if noclipConnection then noclipConnection:Disconnect() end
        end
    end
})

MainTab:Button({
    Title = "Server Hop",
    Callback = function()
        WindUI:Notify({ Title = "Server Hop", Content = "Searching for servers...", Duration = 3 })
        local placeId = game.PlaceId
        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100")).data
        end)
        if success and result then
            for _, s in pairs(result) do
                if s.playing < s.maxPlayers and s.id ~= game.JobId then
                    TeleportService:TeleportToPlaceInstance(placeId, s.id, LocalPlayer)
                    break
                end
            end
        end
    end
})

----------------------------------------------------
-- CHARACTERS TAB
----------------------------------------------------
CharactersTab:Section({ Title = "Yuji Itadori" })

local yujiScriptLoaded = false

CharactersTab:Toggle({
    Title = "Yuji Mode",
    Default = false,
    Callback = function(Value)
        if Value then
            if not yujiScriptLoaded then
                yujiScriptLoaded = true
                pcall(function()
                    loadstring(game:HttpGet("https://pastebin.com/raw/aiy3qDuu"))()
                end)
            end
            WindUI:Notify({ Title = "Yuji Mode", Content = "Đã BẬT Yuji Mode", Duration = 2 })
        else
            WindUI:Notify({ Title = "Yuji Mode", Content = "Đã TẮT Yuji Mode", Duration = 2 })
        end
    end
})

----------------------------------------------------
-- COMBAT TAB
----------------------------------------------------
CombatTab:Section({ Title = "Bypass Anti-Cheat" })

local bypassToggle
bypassToggle = CombatTab:Toggle({
    Title = "Auto Bypass AC",
    Default = false,
    Callback = function(Value)
        bypassACEnabled = Value
        if bypassACEnabled then
            StartAutoBypassAC(bypassToggle)
        else
            if bypassACThread then
                task.cancel(bypassACThread)
                bypassACThread = nil
            end
        end
    end
})

CombatTab:Section({ Title = "Orbit Target" })

local orbitTargetInputText = ""
CombatTab:Input({
    Title = "Mục tiêu Orbit",
    Default = "",
    Placeholder = "Nhập tên người chơi...",
    Callback = function(text)
        orbitTargetInputText = text
    end
})

CombatTab:Toggle({
    Title = "Bật/Tắt Orbit Target",
    Default = false,
    Callback = function(Value)
        orbitEnabled = Value
        if orbitEnabled then
            orbitTargetPlayer = nil
            if orbitTargetInputText ~= "" then
                local name = orbitTargetInputText:lower()
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= LocalPlayer and (p.Name:lower():find(name) or p.DisplayName:lower():find(name)) then
                        orbitTargetPlayer = p
                        break
                    end
                end
            else
                local closestObj = GetClosestTarget()
                if closestObj then orbitTargetPlayer = closestObj end
            end

            if not orbitTargetPlayer or not orbitTargetPlayer.Character or not orbitTargetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                WindUI:Notify({ Title = "Orbit Target", Content = "Không tìm thấy người chơi!", Duration = 2 })
                orbitEnabled = false
                return
            end

            orbitConnection = RunService.RenderStepped:Connect(function(deltaTime)
                if not orbitEnabled or not orbitTargetPlayer or not orbitTargetPlayer.Character or not orbitTargetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    return
                end

                local myChar = LocalPlayer.Character
                if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end

                local targetHrp = orbitTargetPlayer.Character.HumanoidRootPart
                orbitAngle = orbitAngle + (orbitSpeed * deltaTime)
                local offsetX = math.cos(orbitAngle) * orbitDistance
                local offsetZ = math.sin(orbitAngle) * orbitDistance

                local newPos = targetHrp.Position + Vector3.new(offsetX, 0, offsetZ)
                myChar.HumanoidRootPart.CFrame = CFrame.new(newPos, targetHrp.Position)
            end)
        else
            if orbitConnection then
                orbitConnection:Disconnect()
                orbitConnection = nil
            end
        end
    end
})

CombatTab:Slider({
    Title = "Tốc độ Orbit",
    Default = 20,
    Min = 1,
    Max = 100,
    Step = 1,
    Callback = function(Value) orbitSpeed = Value end
})

CombatTab:Slider({
    Title = "Khoảng cách Orbit",
    Default = 5,
    Min = 1,
    Max = 8,
    Step = 0.1,
    Callback = function(Value) orbitDistance = Value end
})

CombatTab:Section({ Title = "Kill All Functions" })

CombatTab:Toggle({
    Title = "0.2 Kill All Gojo Sweep",
    Default = false,
    Callback = function(Value)
        auto02sUltEnabled = Value
        if auto02sUltEnabled then
            local targetIndex = 1
            gojoLoopConnection = RunService.Heartbeat:Connect(function()
                if not auto02sUltEnabled then return end
                local myChar = LocalPlayer.Character
                if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end

                local playerList = {}
                for _, p in ipairs(Players:GetPlayers()) do
                    if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") then
                        if p.Character.Humanoid.Health > 0 then
                            table.insert(playerList, p)
                        end
                    end
                end

                if #playerList > 0 then
                    if targetIndex > #playerList then targetIndex = 1 end
                    local targetPlayer = playerList[targetIndex]
                    if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
                        local targetHrp = targetPlayer.Character.HumanoidRootPart
                        myChar.HumanoidRootPart.CFrame = targetHrp.CFrame * CFrame.new(0, 0, 0.5)
                        
                        pcall(function()
                            ReplicatedStorage.Knit.Knit.Services.GojoService.RE.Ultimate:FireServer()
                            ReplicatedStorage.Knit.Knit.Services.GojoService.RE.RightActivated:FireServer(nil)
                        end)
                    end
                    targetIndex = targetIndex + 1
                end
            end)
        else
            if gojoLoopConnection then
                gojoLoopConnection:Disconnect()
                gojoLoopConnection = nil
            end
        end
    end
})

----------------------------------------------------
-- TP LOCATIONS TAB
----------------------------------------------------
TPTab:Section({ Title = "Địa điểm Teleport" })

local jjsLocations = {
    {"Main Spawn Area", Vector3.new(0, 10, 0)},
    {"Rooftop Area", Vector3.new(-50, 85, -120)},
    {"Park Area", Vector3.new(120, 10, -80)},
    {"School Yard", Vector3.new(-150, 10, 100)},
    {"Destroyed City Zone", Vector3.new(200, 10, 150)},
    {"Safe Sky Zone", Vector3.new(0, 500, 0)}
}

for _, loc in ipairs(jjsLocations) do
    TPTab:Button({
        Title = loc[1],
        Callback = function()
            local char = LocalPlayer.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                char.HumanoidRootPart.CFrame = CFrame.new(loc[2])
                WindUI:Notify({ Title = "Teleport", Content = "Teleported to: " .. loc[1], Duration = 2 })
            end
        end
    })
end

----------------------------------------------------
-- ESP TAB
----------------------------------------------------
ESPTab:Section({ Title = "Cài đặt ESP" })

ESPTab:Toggle({
    Title = "ESP 2D Box",
    Default = false,
    Callback = function(Value) esp2DEnabled = Value end
})

ESPTab:Toggle({
    Title = "ESP Skeleton",
    Default = false,
    Callback = function(Value) skeletonEnabled = Value end
})

----------------------------------------------------
-- FUN TAB
----------------------------------------------------
FunTab:Section({ Title = "Troll / Fun Features" })

FunTab:Toggle({
    Title = "Sóc Lọ",
    Default = false,
    Callback = function(Value)
        socLoDoing = Value
        if socLoDoing then
            socLoThread = task.spawn(function()
                while socLoDoing do
                    local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
                    local humanoid = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid")
                    
                    local rigType = humanoid.RigType == Enum.HumanoidRigType.R15 and "R15" or "R6"
                    local animation = Instance.new("Animation")
                    animation.Name = "aaa"
                    animation.Parent = workspace
                    animation.AnimationId = rigType == "R15" and "rbxassetid://698251653" or "rbxassetid://72042024"

                    local animator = humanoid:FindFirstChildOfClass("Animator") or humanoid:WaitForChild("Animator")
                    if not socLoAnimTrack and animator then
                        socLoAnimTrack = animator:LoadAnimation(animation)
                    end

                    if socLoAnimTrack then
                        socLoAnimTrack:Play()
                        socLoAnimTrack:AdjustSpeed(0.7)
                        socLoAnimTrack.TimePosition = 0.6
                        task.wait(0.1)

                        while socLoDoing and socLoAnimTrack and socLoAnimTrack.TimePosition < 0.7 do
                            task.wait(0.05)
                        end

                        if socLoAnimTrack then
                            socLoAnimTrack:Stop()
                            socLoAnimTrack:Destroy()
                            socLoAnimTrack = nil
                        end
                    end
                    animation:Destroy()
                end
            end)
        else
            if socLoAnimTrack then
                socLoAnimTrack:Stop()
                socLoAnimTrack:Destroy()
                socLoAnimTrack = nil
            end
            if socLoThread then
                task.cancel(socLoThread)
                socLoThread = nil
            end
        end
    end
})

----------------------------------------------------
-- MISC TAB
----------------------------------------------------
MiscTab:Section({ Title = "Tính năng khác" })

MiscTab:Toggle({
    Title = "Invisible",
    Default = false,
    Callback = function(Value)
        invisibleEnabled = Value
        local char = LocalPlayer.Character
        if not char then return end

        if invisibleEnabled then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") or part:IsA("Decal") then
                    originalTransparencies[part] = part.Transparency
                    part.Transparency = 1
                end
            end
        else
            for part, trans in pairs(originalTransparencies) do
                if part and part.Parent then part.Transparency = trans end
            end
            table.clear(originalTransparencies)
        end
    end
})
