-- @Nova.Br Auto Flot v1 GUI (Movable, Rainbow, Foguete, Plataforma "gruda" embaixo do player) + ESP Colorido
local player = game.Players.LocalPlayer
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "NovaBrUI"

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,180,0,56)
main.Position = UDim2.new(0.5,-90,0.12,0)
main.BackgroundColor3 = Color3.fromRGB(26,20,37)
main.BackgroundTransparency = 0.12
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
main.ClipsDescendants = true

local corner = Instance.new("UICorner", main)
corner.CornerRadius = UDim.new(0, 10)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1, 0, 0, 24)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "@Nova.Br"
title.TextColor3 = Color3.fromRGB(170,143,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 16
title.ZIndex = 11

local btn = Instance.new("TextButton", main)
btn.Size = UDim2.new(1,-18,0,22)
btn.Position = UDim2.new(0,9,0,28)
btn.BackgroundColor3 = Color3.fromRGB(67,91,70)
btn.BackgroundTransparency = 0.07
btn.Font = Enum.Font.GothamSemibold
btn.TextColor3 = Color3.fromRGB(255,255,255)
btn.TextSize = 15
btn.Text = "Auto Flot v1 🚀 : OFF"
btn.ZIndex = 11

local btnCorner = Instance.new("UICorner", btn)
btnCorner.CornerRadius = UDim.new(0, 8)

local on = false
local platform = nil
local rainbow = 0

-- FPS ESP Setup
local espFolder = Instance.new("Folder", workspace)
espFolder.Name = "NovaBrESP_Folder"

function createPlatform()
    local char = player.Character or player.CharacterAdded:Wait()
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end
    if not platform then
        platform = Instance.new("Part")
        platform.Size = Vector3.new(7, 1, 7)
        platform.Anchored = true
        platform.CanCollide = true
        platform.Transparency = 0.15
        platform.Name = "NovaBrPlatform"
        platform.Parent = workspace
        local pfcorner=Instance.new("UICorner",platform)
        pfcorner.CornerRadius = UDim.new(0.08,0)
    end
    return root
end

-- ESP FUNC
local espBoxes = {}

function addESP(plr)
    if plr == player then return end
    if espBoxes[plr] then
        if espBoxes[plr].Adornee and espBoxes[plr].Adornee.Parent then
            return -- Already exists
        end
    end
    local function makeBox()
        local char = plr.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            local box = Instance.new("BoxHandleAdornment")
            box.Name = "NovaBrESP"
            box.Adornee = root
            box.AlwaysOnTop = true
            box.ZIndex = 10
            box.Size = Vector3.new(4,7,2)
            box.Transparency = 0.4
            box.Parent = espFolder
            espBoxes[plr] = box
        end
    end
    makeBox()
    plr.CharacterAdded:Connect(function()
        wait(0.3)
        makeBox()
    end)
    plr.AncestryChanged:Connect(function()
        if not plr.Parent and espBoxes[plr] then
            espBoxes[plr]:Destroy()
            espBoxes[plr] = nil
        end
    end)
end

-- Remove ESP when player leaves
game.Players.PlayerRemoving:Connect(function(plr)
    if espBoxes[plr] then
        espBoxes[plr]:Destroy()
        espBoxes[plr] = nil
    end
end)

-- Inicial ESP
for _,plr in ipairs(game.Players:GetPlayers()) do
    addESP(plr)
end
game.Players.PlayerAdded:Connect(addESP)

game:GetService("RunService").RenderStepped:Connect(function()
    rainbow = (rainbow + 0.012) % 1
    main.BackgroundColor3 = Color3.fromHSV((rainbow+0.8)%1, 0.44, 0.17)
    btn.BackgroundColor3 = Color3.fromHSV((rainbow+0.11)%1, 0.28, 0.24)
    title.TextColor3 = Color3.fromHSV((rainbow+0.19)%1,0.34,0.9)
    if on then
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            if not platform or not platform.Parent then
                createPlatform()
            end
            platform.Color = Color3.fromHSV(rainbow,0.7,0.85)
            platform.Position = root.Position - Vector3.new(0, root.Size.Y/2 + 1.5, 0)
        elseif platform then
            platform:Destroy()
            platform = nil
        end
    elseif platform then
        platform:Destroy()
        platform = nil
    end
    
    -- Atualiza cor e posição dos ESPs
    for plr,box in pairs(espBoxes) do
        local char = plr.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if root then
            box.Adornee = root
            box.Size = Vector3.new(4,7,2)
            box.Color3 = Color3.fromHSV((rainbow + (plr.UserId%10)*0.1)%1, 0.85, 1)
            box.Visible = true
        else
            box.Visible = false
        end
    end
end)

btn.MouseButton1Click:Connect(function()
    on = not on
    btn.Text = "Auto Flot v1 🚀 : "..(on and "ON" or "OFF")
    if on then
        createPlatform()
    elseif platform then
        platform:Destroy()
        platform = nil
    end
end)

player.CharacterAdded:Connect(function()
    if platform then platform:Destroy() platform = nil end
    if on then wait(1) createPlatform() end
end)
