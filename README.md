# RepairManGame
:Animation Data:
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;

namespace GladiatorProject
{// Contains a list (so dict doesn't have to be lists)
    public class AnimationData
    {
        /// <summary>
        /// Milliseconds per frame
        /// </summary>
        public int milliseconds { get; set; }
        /// <summary>
        /// Name of the sprite
        /// </summary>
        public string name { get; set; }
        /// <summary>
        /// Rows in the sprite
        /// </summary>
        public int rows { get; set; }
        /// <summary>
        /// Columns in the sprite
        /// </summary>
        public int cols { get; set; }
        /// <summary>
        /// The sprite
        /// </summary>
        public Texture2D sprite { get; set; }
        /// <summary>
        /// Paramaterized Constructor
        /// </summary>
        /// Name
        /// <param name="n"></param>
        /// Rows
        /// <param name="r"></param>
        /// Columns
        /// <param name="c"></param>
        /// SpriteSheet
        /// <param name="s"></param>
        /// Milliseconds per frame
        /// <param name="mpf"></param>
        public AnimationData(string n, int r, int c, Texture2D s, int mpf)
        {
            milliseconds = mpf;
            name = n;
            rows = r;
            cols = c;
            sprite = s;
        }
        
    }
}

:Animation:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
#endregion

namespace GladiatorProject
{
    /// <summary>
    /// Is an attribute of game objects
    /// that contains a dictionary of that 
    /// object's animations
    /// and the number of animations
    /// </summary>
    class Animations
    {
        /// <summary>
        /// Dictionary of animations for the GameObject
        /// </summary>
        public Dictionary<string, AnimationData> sprites = new Dictionary<string, AnimationData>();
        /// <summary>
        /// Number of animations
        /// </summary>
        public int numAnims = 0;
        /// <summary>
        /// Adds an animation to the dictionary of the object's animations
        /// </summary>
        /// <param name="data"></param>
        /// <param name="animName"></param>
        public void AddAnimation(AnimationData data, string animName)
        {
            sprites.Add(animName, data);
            numAnims++;
        }
        /// <summary>
        /// Returns a AnimationData value from the _name key
        /// </summary>
        /// The name key
        /// <param name="_name"></param>
        /// <returns></returns>
        public AnimationData getSprite(string _name)
        {
            // If/else statement that truncates the duplicate index of _name
            string c = "_";
            string[] truncate;
            if (_name.Contains(c) == true)
            {
                truncate = _name.Split('_');
                return sprites[truncate[1]];
            }
            else
                return sprites[_name];
        }
    }
}

:Content Manager:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Audio;
#endregion


namespace GladiatorProject
{
    /// <summary>
    /// The static ContentManager is responsible for taking information parsed and initialized in 
    /// the Game1 class and creating the corresponding game assets.  It will have (at least) attributes for 
    /// animations ( <string, Animations> dictionary) and for sounds (<string, SoundData> dictionary).  Methods in 
    /// this class include (at least) CreateAnimation, CreateSound, and corresponding get methods.  
    /// </summary>
    static class ContentManager
    {
        public static Dictionary<string, Animations> animationDictionary = new Dictionary<string, Animations>();
        public static Dictionary<string, List<SoundEffect>> soundDictionary = new Dictionary<string,List<SoundEffect>>();
        public static Animations tempAnim;
        public static void CreateAnimation(List<Texture2D> _texture, string _name, int numAnims)
        {
            string line;
            string[] tempArray;
            bool canParse;
            List<AnimationData> dataList = new List<AnimationData>();
            AnimationData data;
            int rows, columns, milliseconds, i = 0;
            System.IO.StreamReader file = new System.IO.StreamReader("../../../Content/AnimationData/" + _name + ".txt");
            tempAnim = new Animations();
            while ((line = file.ReadLine()) != null)
            {
                tempArray = line.Split(',');
                canParse = int.TryParse(tempArray[1], out milliseconds);
                canParse = int.TryParse(tempArray[2], out rows);
                canParse = int.TryParse(tempArray[3], out columns);
                data = new AnimationData(tempArray[0], rows, columns, _texture[i], milliseconds);
                i++;
                tempAnim.AddAnimation(data, tempArray[0]);
                numAnims--;
                if (numAnims == 0)
                    animationDictionary.Add(_name, tempAnim);
            }
            file.Close();
        }
        public static void CreateSound(List<SoundEffect> sounds, string _name)
        {
            soundDictionary.Add(_name, sounds);
        }
        public static Animations GetAnimations(string _key)
        {
            return animationDictionary[_key];
        }
        public static List<SoundEffect> GetSounds(string _key)
        {
            return soundDictionary[_key];
        }
    }
}

:Game1:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Audio;
#endregion

namespace GladiatorProject
{
    /// <summary>
    /// (1) Parsing and Loading Data: 
    ///     This data includes a level grid (a text file with x rows and 
    ///     y columns of character indexes for game levels) , texture data (a text file containing the names 
    ///     and types of sprites), and sound data (a text file containing the names and types of game sounds). 
    ///     Each of these files are to be parsed in the LoadContent method.
    /// (2) Loading content:  
    ///     Content Loading is done parallel to parsing data.  The textures and 
    ///     sounds that are initialized in this method are used to set attributes and create data structures in 
    ///     classes that will be mentioned later in this document.  
    /// (3) Calling methods of static classes:
    ///     The Game1 class also calls methods of the static
    ///     classes: ContentManager (CreateAnimation & CreateSound) and ObjectHandler (Draw and Update).
    /// </summary>
    public class Game1 : Game
    {
        GraphicsDeviceManager graphics;
        SpriteBatch spriteBatch;
        /// <summary>
        ///  Current
        /// </summary>

        public static KeyboardState KBState;
        /// <summary>
        ///  Old
        /// </summary>
        public static KeyboardState oldKBState;
        /// <summary>
        ///  Array of levels char[x] (x can be changed based on # of levels)
        /// </summary>
        public char[] levels = new char[25];
        /// <summary>
        /// Dictionary of parsed textures
        /// </summary>
        public Dictionary<string, List<Texture2D>> textures = new Dictionary<string, List<Texture2D>>();
        /// <summary>
        /// Dictionary of parsed sounds
        /// </summary>
        public Dictionary<string, List<SoundEffect>> sounds = new Dictionary<string, List<SoundEffect>>();
        /// <summary>
        /// A Font
        /// </summary>
        SpriteFont devFont;
        /// <summary>
        /// String to hold level chars
        /// </summary>
        string tempFile;
        /// <summary>
        /// Temporary string var
        /// </summary>
        string tempString;
        public Game1()
            : base()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = "Content";
        }


        protected override void Initialize()
        {
            // TODO: Add your initialization logic here
            graphics.PreferredBackBufferWidth = 1080;// set this value to the desired width of your window
            graphics.PreferredBackBufferHeight = 720;// set this value to the desired height of your window
            // Parse LevelGrid.txt
            System.IO.StreamReader file = new System.IO.StreamReader("../../../Content/Levels/LevelGrid.txt");
            tempFile = file.ReadToEnd();
            for (int x = 0; x < 3; x++)
            {
                tempString = tempFile.Substring(x, 1);
                levels[x] = Convert.ToChar(tempString);
            }
            // Set the ObjectHandler's levelIndex
            ObjectHandler.levelIndex = levels;
            file.Close();
            // Apply graphics changes
            graphics.ApplyChanges();
            base.Initialize();
        }

        protected override void LoadContent()
        {
            // Initialize Vars
            string line;
            string[] split = new string[2];
            int numTextures = 0, numSounds = 0;
            string name = "";
            int i = 0, _numTextures = 0, _numSounds = 0;
            List<Texture2D> tempTextures = new List<Texture2D>();
            List<SoundEffect> tempSounds = new List<SoundEffect>();
            List<string> lines = new List<string>();
            List<string[]> splits = new List<string[]>();
            Texture2D tempTexture;
            SoundEffect tempSound;
            string dataString;
            devFont = Content.Load<SpriteFont>("MyFont");
            spriteBatch = new SpriteBatch(GraphicsDevice);
            // Initialize IO Stream
            System.IO.StreamReader file = new System.IO.StreamReader("../../../Content/Textures/TextureData.txt");
            // Parse the TextureData file
            while ((line = file.ReadLine()) != null)
            {
                lines.Add(line);
                if (line.Contains(",") && i == 0)
                {
                    split = line.Split(',');
                    name = split[0];
                    numTextures = int.Parse(split[1]);
                    _numTextures = numTextures;
                    i = 1;
                }
                else if (i == 1 && line.Contains(",") == false)
                {
                    dataString = line;
                    tempTexture = Content.Load<Texture2D>(dataString);
                    tempTextures.Add(tempTexture);
                    if (textures.ContainsKey(name) == false)
                        textures.Add(name, tempTextures);
                    _numTextures--;
                    if (_numTextures == 0)
                    {
                        ContentManager.CreateAnimation(textures[name], name, numTextures);
                        tempTextures.Clear();
                        i = 0;
                    }
                }
            }
            file.Close();
            lines.Clear();
            // Initialize IO stream
            System.IO.StreamReader file2 = new System.IO.StreamReader("../../../Content/SoundData/SoundData.txt");
            // Parse the SoundData file
            while ((line = file2.ReadLine()) != null)
            {
                lines.Add(line);
                if (line.Contains(",") && i == 0)
                {
                    split = line.Split(',');
                    name = split[0];
                    numSounds = int.Parse(split[1]);
                    _numSounds = numSounds;
                    i = 1;
                }
                else if (i == 1 && line.Contains(",") == false)
                {
                    dataString = line;
                    tempSound = Content.Load<SoundEffect>(dataString);
                    tempSounds.Add(tempSound);
                    if (sounds.ContainsKey(name) == false)
                        sounds.Add(name, tempSounds);
                    _numSounds--;
                    if (_numSounds == 0)
                    {
                        ContentManager.CreateSound(sounds[name], name);
                        tempTextures.Clear();
                        i = 0;
                    }
                }
            }
            file2.Close();
            // Initialize Keyboard states
            KBState = new KeyboardState();
            oldKBState = new KeyboardState();
        }

        protected override void UnloadContent()
        {

        }
        protected override void Update(GameTime gameTime)
        {

            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed || Keyboard.GetState().IsKeyDown(Keys.Escape))
                Exit();
            // Update the current keyboard state
            KBState = Keyboard.GetState();
            // Call the update function passing it gameTime
            ObjectHandler.Update(gameTime);
            // Update the old keyboard state
            oldKBState = KBState;

            base.Update(gameTime);
        }


        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);
            // Call draw function passing it spriteBatch
            ObjectHandler.Draw(spriteBatch);
            base.Draw(gameTime);
        }


    }
}

:GameObject:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Audio;
#endregion

namespace GladiatorProject
{

    /// <summary>
    /// Possible Attributes...
    ///     string name
    ///     vector2 position
    ///     rectangle hitBox
    ///     List<SoundEffect> sounds
    ///     Position in the game grid
    ///     Texture2D sprite
    ///     AnimationData currentAnimation
    ///     Animations Animation
    ///     bool canCollide
    ///     bool onScreen
    ///     ...
    /// 
    /// Possible Methods...
    ///     Paramaterized Constructor
    ///     Update
    ///     Draw
    ///     ...
    /// </summary>
    class GameObject
    {
        string name;
        Vector2 position;
        Rectangle hitBox;
        List<SoundEffect> sounds;
        Texture2D sprite;
        AnimationData currentAnimation;
        Animations animation;
        bool canCollide;
        bool onScreen;

        public GameObject(string nm, Vector2 pstn, Animations anmtn, Texture2D sprt, bool cnClld, bool nScrn)
        {
            //parameters
            name = nm;
            position = pstn;
            animation = anmtn;
            sprite = sprt;

            //Initialize a starting animation, and a hitbox based upon parameters
            currentAnimation = animation.getSprite(name);
            hitBox = new Rectangle((int)position.X, (int)position.Y, sprite.Width, sprite.Height);

            //more parameters
            canCollide = cnClld;
            onScreen = nScrn;
            sounds = ContentManager.GetSounds(name);
        }

        public virtual void Update()
        {

        }

        public virtual void Draw()
        {

        }

    }
}

:Level:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
#endregion

namespace GladiatorProject
{
    class Level
    {
        const int numLevels = 1;
        string line;
        Dictionary<string, GameObject> objectDictionary;
        string tempX, tempY;
        GameObject obj;
        Vector2 vector;
        char type;
        string name;
        public char[][] levelArray;
        int x = 0;

        bool canParse;
        public Level()
        {
            objectDictionary = new Dictionary<string, GameObject>();
        }

        public char[][] GetLevel()
        {
            return levelArray;
        }

        public void LoadLevel(char _index)
        {
            System.IO.StreamReader file = new System.IO.StreamReader("../../../Content/Levels/Level" + _index + ".txt");
            Initialize(file);
        }
        public void Initialize(System.IO.StreamReader _file)
        {
            while ((line = _file.ReadLine()) != null)
            {
               // Read level file here
                levelArray[x] = line.ToCharArray();

                for (int i = 0; i < levelArray.Length; i++)
                {
                    type = levelArray[x][i];
                    // Switch statement for character index "type"
                    switch (type)
                    {
                        case '-':
                            //the top of ground
                            break;
                        case 'p':
                            //the player
                            break;
                        case '!':
                            //a melee enemy
                            break;
                        case 'o':
                            //the solid wall of ground
                            break;
                        case '_':
                            //a platform
                            break;
                        case 'l':
                            //a ranged enemy
                            break;
                        case 's':
                            //a switch
                            break;
                        case 'd':
                            //a door
                            break;
                        case '^':
                            //a bounce platform
                            break;
                        default:
                            //obj = new GameObject(name, vector, ContentManager.GetAnimations(type));
                            break;
                    }
                }
                x++;

                ObjectHandler.AddToObjectDictionary(obj, name);
            }
            _file.Close();
        }

    }
}

:MovingGameObject:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
#endregion

namespace GladiatorProject
{
    /// <summary>
    /// 
    /// </summary>
    class MovingGameObject : GameObject
    {
        public MovingGameObject(string nm, Vector2 pstn, Animations anmtn, Texture2D sprt, bool cnClld, bool nScrn): base(nm, pstn, anmtn, sprt, cnClld, nScrn)
        {

        }

        public virtual void Move()
        {

        }

        public virtual void Update()
        {

        }

        public virtual void Draw()
        {

        }
    }
}

:ObjectHandler:
#region Using Statements
using System;
using System.Collections.Generic;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Media;
using Microsoft.Xna.Framework.Storage;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Audio;
#endregion

namespace GladiatorProject
{
    /// <summary>
    /// The static ContentManager is responsible for taking information parsed and initialized in 
    /// the Game1 class and creating the corresponding game assets.  It will have (at least) attributes for 
    /// animations ( <string, Asset> dictionary) and for sounds (<string, Asset> dictionary).  Methods in 
    /// this class include (at least) CreateAnimation, CreateSound, and corresponding get methods.  
    /// These collections are not of on-screen objects but of possible on-screen objects.
    /// </summary>
    static class ObjectHandler
    {
        
        public static int[,] collisionGrid = new int[10, 10];
        public static Level currentLevel;
        public static bool collision = false;
        static bool initialUpdate = true;
        public static char[] levelIndex { get; set; }
        /// <summary>
        /// -1: previous
        /// 0: current
        /// 1: next
        /// </summary>
        public static int changeLevel = 0;
        public static Dictionary<string, GameObject> objectDictionary = objectDictionary = new Dictionary<string,GameObject>();
        public static List<string> onScreen = new List<string>();
        public static List<SoundEffect> sounds = new List<SoundEffect>();
        public static int index { get; set; }
        
        public static void AddToObjectDictionary(GameObject value, string key)
        {
            string c = "_";
            int duplicateIndex = 2;
            while(true)
            { 
                if(objectDictionary.ContainsKey(key) == false)
                {
                    objectDictionary.Add(key, value);
                    onScreen.Add(key);
                    //value.name = key;
                    return;
                }
                else
                {
                    if(key.Contains(c))
                    {
                        key = key.Substring(key.IndexOf('_')+1);
                    }
                    key = duplicateIndex.ToString() + "_" + key;
                    duplicateIndex++;
                }
            }
        }
        /// <summary>
        /// Updates each object on screen
        /// </summary>
        public static void Update(GameTime gameTime)
        {
            if(initialUpdate == true)
            {
                index = 0;
                InitializeLevel();
                initialUpdate = false;
            }
            if (changeLevel == 1)
            {
                index++;
                InitializeLevel();
                changeLevel = 0;
            }
            if (changeLevel == -1)
            {
                index--;
                InitializeLevel();
                changeLevel = 0;
            }
            if (changeLevel == 0)
            {
                for (int x = 0; x < objectDictionary.Count; x++)
                {
                    //objectDictionary[onScreen[x]].Update(gameTime);
                }
            }
        }
        /// <summary>
        /// Draws each object on screen
        /// </summary>
        public static void Draw(SpriteBatch spriteBatch)
        {
            for(int x = 0; x < objectDictionary.Count; x++)
            {
                //objectDictionary[onScreen[x]].Draw(spriteBatch);
            }
        }
        public static void InitializeLevel()
        {

            for (int x = 0; x < objectDictionary.Count; x++)
            {
                //objectDictionary[onScreen[x]].StopSounds();
            }
            ClearLevel();
            currentLevel = new Level();
            currentLevel.LoadLevel(levelIndex[index]);
        }
        public static void ClearLevel()
        {
            onScreen.Clear();
            objectDictionary.Clear();

            for (int x = 0; x < 10; x++)
            {
                for (int y = 0; y < 10; y++)
                {
                    collisionGrid[x, y] = 0;
                }
            }
        }
        public static bool AddCollision(GameObject obj, Rectangle hB)
        {
            Rectangle _hB;
            GameObject _obj;
            for(int i = 0; i < objectDictionary.Count; i++)
            {
                //_hB = objectDictionary[onScreen[i]].GetHitBox();
                _obj = objectDictionary[onScreen[i]];
                //if (hB.Intersects(_hB) && obj != _obj && _obj.canCollide == true)
                {
                    //obj.SetCollision(_hB);
                    return true;
                }
            }
            return false;
        }
        public static GameObject GetCollision(int x, int y)
        {
            return null;
        }
    }
}

:Program:
#region Using Statements
using System;
using System.Collections.Generic;
using System.Linq;
#endregion

namespace GladiatorProject
{
#if WINDOWS || LINUX
    /// <summary>
    /// The main class.
    /// </summary>
    public static class Program
    {
        /// <summary>
        /// The main entry point for the application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            using (var game = new Game1())
                game.Run();
        }
    }
#endif
}
