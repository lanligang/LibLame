# LibLame
音频压缩转码文件 ： lame
> 使用方法
```
- (void)conventToMp3 {

NSString *cafFilePath = [NSTemporaryDirectory() stringByAppendingString:@"VoiceInputFile"];
_mp3FileName = [[NSUUID UUID] UUIDString];
self.mp3FileName = [self.mp3FileName stringByAppendingString:@".mp3"];
NSString *mp3FilePath = [[NSHomeDirectory() stringByAppendingFormat:@"/Documents/"] stringByAppendingPathComponent:self.mp3FileName];

@try {

int read, write;

FILE *pcm = fopen([cafFilePath cStringUsingEncoding:NSASCIIStringEncoding], "rb");
FILE *mp3 = fopen([mp3FilePath cStringUsingEncoding:NSASCIIStringEncoding], "wb");

const int PCM_SIZE = 8192;
const int MP3_SIZE = 8192;
short int pcm_buffer[PCM_SIZE * 2];
unsigned char mp3_buffer[MP3_SIZE];

lame_t lame = lame_init();
lame_set_in_samplerate(lame, self.sampleRate);
lame_set_VBR(lame, vbr_default);
lame_init_params(lame);

long curpos;
BOOL isSkipPCMHeader = NO;

do {

curpos = ftell(pcm);

long startPos = ftell(pcm);

fseek(pcm, 0, SEEK_END);
long endPos = ftell(pcm);

long length = endPos - startPos;

fseek(pcm, curpos, SEEK_SET);


if (length > PCM_SIZE * 2 * sizeof(short int)) {

if (!isSkipPCMHeader) {
//Uump audio file header, If you do not skip file header
//you will heard some noise at the beginning!!!
fseek(pcm, 4 * 1024, SEEK_SET);
isSkipPCMHeader = YES;
NSLog(@"skip pcm file header !!!!!!!!!!");
}

read = (int)fread(pcm_buffer, 2 * sizeof(short int), PCM_SIZE, pcm);
write = lame_encode_buffer_interleaved(lame, pcm_buffer, read, mp3_buffer, MP3_SIZE);
fwrite(mp3_buffer, write, 1, mp3);
NSLog(@"read %d bytes", write);
}

else {

[NSThread sleepForTimeInterval:0.05];
NSLog(@"sleep");

}

} while (!self.isStopRecorde);

read = (int)fread(pcm_buffer, 2 * sizeof(short int), PCM_SIZE, pcm);
write = lame_encode_flush(lame, mp3_buffer, MP3_SIZE);
DWDLog(@"read %d bytes and flush to mp3 file", write);

lame_close(lame);
fclose(mp3);
fclose(pcm);

self.isFinishConvert = YES;
}
@catch (NSException *exception) {
DWDLog(@"%@", [exception description]);
}
@finally {
NSLog(@"convert mp3 finish!!!");
}
}
```
