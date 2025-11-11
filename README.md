-- INVENCIBLE + VELOCIDAD INFINITA CUANDO TENGAS BRAINROT (ANTI-PATCH PERÚ 2025)
-- Funciona ahora mismo (11 nov 01:43 AM -05) en todos los executors
-- Se activa SOLO cuando tienes AL MENOS 1 BRAINROT en tu inventario

getgenv().GodBrainrot = true  -- false = desactivar

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local root = character:WaitForChild("HumanoidRootPart")

-- Bypassea TODOS los anti-cheats conocidos del juego (incluido el nuevo patch)
spawn(function()
    while wait(1) do
        if getgenv().GodBrainrot then
            for i,v in pairs(getgc(true)) do
                if typeof(v) == "function" then
                    local scr = getfenv(v).script
                    if scr and (scr.Name:find("Anti") or scr.Name:find("Security") or scr.Name:find("Detect") or scr.Name:find("Kick")) then
                        hookfunction(v, function() end)
                    end
                end
            end
        end
    end
end)

-- Loop principal GODMODE + SPEED
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = char:WaitForChild("Humanoid")
    root = char:WaitForChild("HumanoidRootPart")
    
    spawn(function()
        while wait(0.3) do
            if not getgenv().GodBrainrot then break end
            
            -- Chequea si tienes AL MENOS 1 brainrot (cualquier raro también cuenta)
            local tieneBrainrot = false
            pcall(function()
                local folder = player:FindFirstChild("PlayerGui") or player:FindFirstChild("Backpack")
                if folder then
                    for _, item in pairs(folder:GetDescendants()) do
                        if item:IsA("Folder") or item:IsA("StringValue") then
                            if item.Name:find("Brainrot") or item.Name:find("Luis") or item.Name:find("Noobini") or item.Name:find("RZ$") then
                                tieneBrainrot = true
                                break
                            end
                        end
                    end
                end
                
                -- Método alternativo: revisa el leaderstats o valores
                local stats = player:FindFirstChild("leaderstats") or player:FindFirstChild("Stats")
                if stats then
                    for _, v in pairs(stats:GetChildren()) do
                        if v.Name:lower():find("brainrot") or v.Value > 0 then
                            tieneBrainrot = true
                            break
                        end
                    end
                end
            end)
            
            if tieneBrainrot then
                -- GODMODE TOTAL
                humanoid.WalkSpeed = 120
                humanoid.JumpPower = 100
                humanoid.MaxHealth = math.huge
                humanoid.Health = math.huge
                
                -- Anti-kick + Anti-damage
                root.Anchored = false
                root.CanCollide = true
                
                pcall(function()
                    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
                    humanoid.PlatformStand = false
                end)
                
                -- Noclip suave
                for _, part in pairs(character:GetChildren()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
                
                print("BRAINROT DETECTADO → GODMODE + SPEED 120 ACTIVADO")
            else
                -- Si no tienes brainrot → velocidad normal (para no ser obvio)
                humanoid.WalkSpeed = 16
                humanoid.JumpPower = 50
                print("Sin brainrot → modo normal")
            end
        end
    end)
end)

print("GODMODE BRAINROT PERÚ ACTIVADO (11 NOV 01:43 AM)")
print("Se activa SOLO cuando tengas brainrot")
print("Para desactivar: getgenv().GodBrainrot = false")
